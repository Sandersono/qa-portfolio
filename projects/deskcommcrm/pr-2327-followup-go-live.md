# PR #2327: Go-Live Hold Does Not Hold Automatic Follow-Ups

**TL;DR:** New WhatsApp numbers start in a "go-live" hold that must stop messages the business starts, to reduce the risk of a ban. Automatic follow-ups ignored the hold, because the rule looked for a key that follow-up jobs never carry. The existing test passed only because its test data used a format no real producer creates.
I fixed the matching rule and added 2 database invariant cases. The maintainer wrote that the fix goes in as is. The PR is open, waiting for the maintainer to confirm a 2.0.0 release.

Related documents: [Overview](overview.md) · [PR #1473](pr-1473-ai-agent-publishing.md) · [PR #2324](pr-2324-webhook-retention.md)

---

## Summary

| Field | Value |
|---|---|
| PR | [melgarafael/DeskcommCRM#2327](https://github.com/melgarafael/DeskcommCRM/pull/2327) |
| Status | Open (October 2026). Accepted as is, waiting for the maintainer to confirm version 2.0.0. |
| Area | AI agent engine: session watchdog and job queue |
| My commit | `eaebb7b79`: 2 files, +104 / −6 |
| Severity (my assessment) | High. The protection for new numbers did not work for automatic follow-ups, which raises the risk of a WhatsApp ban. |

## Defect report

**Title:** Automatic follow-ups are sent while the WhatsApp number is in the go-live hold.

**Environment:** Any installation with a WhatsApp number in the go-live hold (also when the number is in the `block_rate` state).

**Steps to reproduce:**
1. Connect a new WhatsApp number. It starts in the go-live hold.
2. Have a conversation on that number with an automatic follow-up scheduled by the follow-up engine.
3. Wait until the follow-up job runs.

**Expected result:** The follow-up job is held until someone releases the number.

**Actual result:** The follow-up is sent anyway, which is the opposite of what the hold promises.

## Root cause

The hold rule (`enforceHolds` in the session watchdog) only held jobs whose payload had `channel_session_id` equal to the number's ID. That key is used by inbound reply jobs.

The producers of follow-up jobs (the follow-up engine, the scheduler and the API routes that queue the job) do not write that key. They carry the conversation in `payload.service_boundary.conversation_id`, and the follow-up handler finds the channel through it. So follow-up jobs never matched the rule.

**Why the tests did not catch it:** the existing invariant test queued a follow-up job **with** `channel_session_id` in the payload, a format that no real producer creates. The test was green for the wrong reason.

## Fix

The hold now finds a job's channel the same way the handler does: from `channel_session_id` in the payload, or from the conversation in `service_boundary.conversation_id`.

- The conversation ID is compared as text on purpose. Casting a bad value to `uuid` would throw an error for the whole run, and the watchdog would stop holding any job.
- The channel used by the hold is saved on the job (`held_channel_session_id`), and the release matches by it. Release removes both marks, so the job comes back exactly as it was queued.
- Jobs held before this PR only have `channel_session_id`. The release falls back to it, so updating needs no manual database work.
- A job with no channel anywhere still does not match. It fails on its own in the handler, which would not know where to send it either.

## Regression tests

Two new cases in the existing invariant test `tests/invariants/hold-de-go-live-nao-cala-resposta.test.ts`, run with `pnpm test:db` against a real PostgreSQL database:

| # | Case | Expected | Result on old `main` |
|---|---|---|---|
| 1 | Follow-up job queued the way the engine queues it (only `service_boundary`), number in go-live | Held, then released clean | Fails |
| 2 | Follow-up job for a conversation on **another**, healthy number | Not held | Guards against holding too much |

The invariant folder is frozen in this project. No existing assertion was changed or relaxed; I only added cases and the setup they need. I ran the file (7 tests), lint, the type check and the neighboring tests.

## Independent verification (from the maintainer's review)

- Counted the follow-up producers in 8 files: none writes `channel_session_id`. The search itself was checked against a file where the key does exist. The maintainer also found one more producer with the same format.
- The 11 old assertions of the invariant are identical, and 6 new ones were added. In CI, the invariant suite passed on PostgreSQL 15 and 17, with both a fresh install and an update of the database baseline.
- Checked that a held job does not block the rest of the queue for that contact.

## Correction from the review

My PR description said follow-ups were also sent when the channel was down. The maintainer showed that, in that case, the send returned `queued` and the job was rescheduled instead. The real fix applies to the go-live and `block_rate` states. I list this because it is a fair correction of my analysis, not of the fix.

## Why it is not merged yet

The fix makes the hold work for the first time. Many installations have numbers that stayed in go-live for weeks while working normally, because the hold never held anything. After an update, all their automatic follow-ups would stop.

The maintainer decided that my fix goes in as is, and added a transition on top of it:
- a migration that releases numbers that are already established,
- held follow-ups released spaced out, one every 5 minutes per number,
- a warning in the admin center, in Portuguese, Spanish and English.

Because operators may need to act after updating, the project's versioning rule makes this a major version (2.0.0). The PR is ready, and auto-merge is off on purpose until the maintainer confirms that version. All CI checks that ran passed.

## What this shows

- Finding a defect hidden by a test whose data did not match production
- Reading several producers and consumers to find a key mismatch
- Database-level invariant tests, including a test that the fix does not hold too much
- Thinking about rollout risk, not only the code change

[← Back to portfolio](../../README.md)
