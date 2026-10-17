# PR #2324: Webhook Cleanup Silently Stops ("414 URI Too Long")

**TL;DR:** A scheduled job that empties old webhook logs stopped working without any visible error. It sent up to 500 IDs in the URL, the API gateway rejected the request as too long, and the job picked the same batch again every run.
I changed the job to filter by date instead of by ID list and added 4 unit tests. The maintainer broke the code on purpose four times, and each time one of my tests failed. The PR was merged.

Related documents: [Overview](overview.md) · [PR #1473](pr-1473-ai-agent-publishing.md) · [PR #2327](pr-2327-followup-go-live.md)

---

## Summary

| Field | Value |
|---|---|
| PR | [melgarafael/DeskcommCRM#2324](https://github.com/melgarafael/DeskcommCRM/pull/2324) |
| Status | Merged into `main` on 2026-10-05 (opened the same day) |
| Area | Data retention job (`webhook-log-retention`) |
| My commit | `5176e251a`: 3 files, +128 / −20 |
| Severity (my assessment) | High. The largest table in the database keeps growing, and the failure is silent: only a warning in the log. |

## Defect report

**Title:** The webhook log cleanup job never empties any row once a few hundred expired rows exist.

**Environment:** Any installation with a few hundred expired webhook rows, for example after updating an installation that already has history. The API gateway in front of PostgREST has a URL size limit of 8,192 bytes.

**Steps to reproduce:**
1. Have more than a few hundred webhook log rows older than the retention period.
2. Let the `webhook-log-retention` scheduled job run.
3. Check the `archived_at` column of the oldest rows.
4. Let the job run again.

**Expected result:** The job empties the old rows, sets `archived_at`, and the next run moves on to the next batch.

**Actual result:** `archived_at` is never set. The next run picks the same batch. The log only shows a warning ("não consegui esvaziar o lote", "could not empty the batch"), and the job response does not show that cleanup stopped.

## Root cause

The job selected up to 500 expired rows and then updated them with `.update(...).in("id", ids)`. PostgREST puts this filter in the URL query string. 500 UUIDs make a URL longer than 18 KB, and the gateway rejects it with `414 URI Too Long`. The project already documented the same 8,192-byte limit in another API handler.

Because the update failed, `archived_at` was never written. The job always chooses "the oldest rows where `archived_at` is null", so it chose the same rows again, forever.

## Fix

The update now filters by date instead of by a list of IDs: it uses the `received_at` of the last row in the batch, with the same `archived_at is null` condition. The URL now carries only two dates.

Boundary details I handled:
- **Timestamp precision:** the date is kept exactly as the database returns it, with microseconds. Converting it to a JavaScript `Date` would cut it to milliseconds and leave the last row out.
- **Ties:** rows with the exact same timestamp as the last row can be emptied together. This is acceptable, because the job stays idempotent and bounded.
- **Honest reporting:** the job now counts the rows the database actually returned, not the rows it asked for. Otherwise the report would be wrong in the tie case.

## Regression tests

`tests/unit/retencao-do-arquivo-de-webhook.test.ts`, 4 new cases. All 4 fail on the old `main` and pass with the fix:

| # | What the test checks |
|---|---|
| 1 | The update never sends a list of IDs. It also asserts that the update happens, so the test cannot pass by doing nothing. |
| 2 | The cut uses the last row's date, without losing microseconds. |
| 3 | The update repeats the `archived_at is null` condition. |
| 4 | The result counts what the database returned. |

I also updated two existing test files so their fake data includes `received_at`. No existing assertion was changed.

## Independent verification (from the maintainer's review)

- The merge preview with `main` was clean, and the 2 test files in the PR passed: 24 of 24.
- **Mutation check:** the maintainer broke the source code one point at a time. They switched back to the ID list, cut by the first row instead of the last, removed `archived_at is null` from the update, and counted the requested rows instead of the returned rows. All four changes made a test fail. In the maintainer's words, the new tests guard what they say they guard.
- Security check: the cut date always comes from a row that is already expired, so the update only reaches expired rows.

## What the review added

- With the date cut, the selection's sort order now controls the size of each update. The maintainer added a test that locks the ascending order.
- CI found a third test file whose test double did not know the new date filter. My PR did not update it, and the maintainer did. After that, the three retention test files passed: 34 of 34.

**Lesson:** when I change a shared function, I now search every test double that fakes it, not only the tests next to the code.

## What this shows

- Finding a silent failure that only shows up as a warning
- Boundary analysis: URL size limit and timestamp precision
- Tests that were proven useful by mutation (breaking the code on purpose)

[← Back to portfolio](../../README.md)
