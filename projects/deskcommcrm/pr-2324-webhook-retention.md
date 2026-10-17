# PR #2324: Webhook Retention and Oversized URLs

**TL;DR:** A scheduled job that empties old webhook logs stopped working without any visible error. It sent up to 500 IDs in the URL, the API gateway rejected the request as too long, and the job picked the same batch again every run. I changed the job to filter by date instead of by ID list and added 4 unit tests. The maintainer broke the code on purpose four times, and each time one of the tests failed. The PR was merged.

| Field | Value |
|---|---|
| Upstream PR | [melgarafael/DeskcommCRM#2324](https://github.com/melgarafael/DeskcommCRM/pull/2324) |
| Status | **Merged** into `main` on 2026-10-05 (opened the same day) |
| Area | Data retention job (`webhook-log-retention`) |
| My commit | `5176e251a`: 3 files, +128 / −20 |
| Severity (my assessment) | High. The largest table in the database keeps growing, and the failure is silent: only a warning in the log. |

## Defect

A retention job selected up to 500 expired webhook rows and then updated them with a PostgREST filter containing all row IDs.

With hundreds of UUIDs, the query string could exceed the gateway's URL limit and be rejected with **414 URI Too Long**.

The failure was operationally dangerous because:

- `archived_at` was never written;
- the next run selected the same batch again;
- raw webhook bodies continued accumulating;
- the failure surfaced only as a warning, so the cron itself could appear to have run.

### Defect report

**Environment:** Any installation with a few hundred expired webhook rows, for example after updating an installation that already has history. The API gateway in front of PostgREST has a URL size limit of 8,192 bytes.

**Steps to reproduce:**
1. Have more than a few hundred webhook log rows older than the retention period.
2. Let the `webhook-log-retention` scheduled job run.
3. Check the `archived_at` column of the oldest rows.
4. Let the job run again.

**Expected result:** The job empties the old rows, sets `archived_at`, and the next run moves on to the next batch.

**Actual result:** `archived_at` is never set. The next run picks the same batch. The log only shows a warning ("não consegui esvaziar o lote", "could not empty the batch"), and the job response does not show that cleanup stopped.

## Root cause

The implementation used a large `.in("id", ids)` filter. The repository already documented an 8,192-byte gateway limit in another path. A batch of 500 UUIDs could create a URL above 18 KB.

The problem was therefore not the number of rows in the database operation itself. It was the **transport representation of the filter**.

## Fix and regression design

The update was changed to use the `received_at` boundary of the last selected row instead of sending hundreds of IDs in the URL.

Important edge cases included:

- preserving database microseconds instead of converting the boundary through JavaScript `Date`, which would cut it to milliseconds and leave the last row out;
- repeating the `archived_at is null` predicate;
- allowing rows tied at the boundary timestamp while keeping the operation idempotent;
- reporting the number of rows actually returned by the update, not the number originally requested.

### Regression tests

`tests/unit/retencao-do-arquivo-de-webhook.test.ts`, 4 new cases. All 4 fail on the old `main` and pass with the fix:

| # | What the test checks |
|---|---|
| 1 | The update never sends a list of IDs. It also asserts that the update happens, so the test cannot pass by doing nothing. |
| 2 | The cut uses the last row's date, without losing microseconds. |
| 3 | The update repeats the `archived_at is null` condition. |
| 4 | The result counts what the database returned. |

Two existing test files were updated so their fake data includes `received_at`. No existing assertion was changed.

## Independent validation

From the maintainer's public review:

- The merge preview with `main` was clean, and the 2 test files in the PR passed: 24 of 24.
- **Challenge testing (mutation):** the maintainer broke the source code one property at a time. They switched back to the ID list, cut by the first row instead of the last, removed `archived_at is null` from the update, and counted the requested rows instead of the returned rows. All four changes made a test fail, so the tests were not passing vacuously.
- Security check: the cut date always comes from a row that is already expired, so the update only reaches expired rows.

## What the review added

- With the date cut, the selection's sort order now controls the size of each update. The maintainer added a test that locks the ascending order.
- CI found a third test file whose test double did not know the new date filter. My PR did not update it, and the maintainer did. After that, the three retention test files passed: 34 of 34.

Lesson I took from this: when I change a shared function, I search every test double that fakes it, not only the tests next to the code.

## QA lesson

A background job can "run" and still fail its purpose. For operational workflows, QA must test the **observable outcome and failure mode**, not only whether the scheduler invoked the code.

---

[← DeskcommCRM overview](overview.md) · [← Portfolio](../../README.md)
