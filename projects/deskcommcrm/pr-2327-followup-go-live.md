# PR #2327: Go-Live Hold and Real Production Payloads

**TL;DR:** New WhatsApp numbers start in a "go-live" hold that must stop messages the business starts, to reduce the risk of a ban. Automatic follow-ups ignored the hold, because the rule looked for a key that follow-up jobs never carry. The existing test passed only because its test data used a format no real producer creates. I fixed the matching rule and added 2 database invariant cases. The maintainer accepted the core fix; the PR is open, waiting for the 2.0.0 rollout.

| Field | Value |
|---|---|
| Upstream PR | [melgarafael/DeskcommCRM#2327](https://github.com/melgarafael/DeskcommCRM/pull/2327) |
| Status | **Open — core fix accepted by the maintainer; rollout held for 2.0.0** (October 2026) |
| Area | AI agent engine: session watchdog and job queue |
| My commit | `eaebb7b79`: 2 files, +104 / −6 |
| Severity (my assessment) | High. The protection for new numbers did not work for automatic follow-ups, which raises the risk of a WhatsApp ban. |

## Defect

The product had a go-live hold intended to retain automated `followup_turn` jobs for a new channel.

An existing invariant test passed, suggesting the protection worked. Production producers, however, did not emit the field that the hold rule was checking.

The rule looked for `payload.channel_session_id`, while real follow-up producers carried the conversation identity inside `payload.service_boundary.conversation_id`.

The test fixture represented a payload shape that production did not produce.

### Defect report

**Environment:** Any installation with a WhatsApp number in the go-live hold (also when the number is in the `block_rate` state).

**Steps to reproduce:**
1. Connect a new WhatsApp number. It starts in the go-live hold.
2. Have a conversation on that number with an automatic follow-up scheduled by the follow-up engine.
3. Wait until the follow-up job runs.

**Expected result:** The follow-up job is held until someone releases the number.

**Actual result:** The follow-up is sent anyway, which is the opposite of what the hold promises.

## Investigation

The important QA step was not adding another assertion to the same fixture. It was checking the fixture against the actual producers.

The upstream review confirmed that the inspected `followup_turn` producers did not write `channel_session_id`:

- The maintainer counted the producers in 8 files, and none writes that key. The search itself was checked against a file where the key does exist.
- The maintainer also found an additional producer and reached the same result.

This explained how a green invariant could coexist with a production behavior gap.

## Core fix and regression coverage

The proposed fix resolved the channel using the same information path used by the real follow-up handler and retained the resolved channel identity for later release:

- The channel comes from `channel_session_id` in the payload, or from the conversation in `service_boundary.conversation_id`.
- The conversation ID is compared as text on purpose. Casting a bad value to `uuid` would throw an error for the whole run, and the watchdog would stop holding any job.
- The channel used by the hold is saved on the job (`held_channel_session_id`), and the release matches by it. Release removes both marks, so the job comes back exactly as it was queued.
- Jobs held before this PR only have `channel_session_id`. The release falls back to it, so updating needs no manual database work.

Two new cases in the existing invariant test `tests/invariants/hold-de-go-live-nao-cala-resposta.test.ts`, run with `pnpm test:db` against a real PostgreSQL database:

| # | Case | Expected | Result on old `main` |
|---|---|---|---|
| 1 | Follow-up job queued the way the engine queues it (only `service_boundary`), number in go-live | Held, then released clean | Fails |
| 2 | Follow-up job for a conversation on **another**, healthy number | Not held | Guards against holding too much |

The invariant folder is frozen in this project. No existing assertion was changed or relaxed. In the maintainer's check, the 11 old assertions were identical and 6 new ones were added. In CI, the invariant suite passed on PostgreSQL 15 and 17, with both a fresh install and an update of the database baseline.

The maintainer explicitly accepted the core fix.

## Correction from the review

My PR description said follow-ups were also sent when the channel was down. The maintainer showed that, in that case, the send returned `queued` and the job was rescheduled instead. The real fix applies to the go-live and `block_rate` states. This was a fair correction of my analysis, not of the fix.

## Why the PR is still open

Making the hold work correctly changes behavior for existing installations. Older installations can contain go-live state that historically did not block these follow-ups. Enforcing the corrected rule immediately could therefore retain automated messages for already-operating numbers.

The upstream maintainers added migration and rollout work on top of the original finding and fix, including transition rules and additional invariants. The project classifies the behavior change as requiring the **2.0.0** release, so auto-merge is intentionally disabled.

The later migration/rollout commits are maintainer work and are not presented here as my implementation.

## QA lesson

**A passing test is not evidence of production correctness when the fixture is unrealistic.**

Test data must be traced back to real producers, contracts or observed payloads. In this case, validating the fixture itself exposed the defect that the existing invariant could not see.

---

[← DeskcommCRM overview](overview.md) · [← Portfolio](../../README.md)
