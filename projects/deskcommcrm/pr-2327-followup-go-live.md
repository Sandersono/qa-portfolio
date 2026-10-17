# PR #2327: Go-Live Hold and Real Production Payloads

Upstream PR: [melgarafael/DeskcommCRM#2327](https://github.com/melgarafael/DeskcommCRM/pull/2327)  
Status: **Open — core fix accepted by the maintainer; rollout held for 2.0.0**

## Defect

The product had a go-live hold intended to retain automated `followup_turn` jobs for a new channel.

An existing invariant test passed, suggesting the protection worked. Production producers, however, did not emit the field that the hold rule was checking.

The rule looked for `payload.channel_session_id`, while real follow-up producers carried the conversation identity inside `payload.service_boundary.conversation_id`.

The test fixture represented a payload shape that production did not produce.

## Investigation

The important QA step was not adding another assertion to the same fixture. It was checking the fixture against the actual producers.

The upstream review confirmed that the inspected `followup_turn` producers did not write `channel_session_id`. The maintainer also found an additional producer and reached the same result.

This explained how a green invariant could coexist with a production behavior gap.

## Core fix and regression coverage

The proposed fix resolved the channel using the same information path used by the real follow-up handler and retained the resolved channel identity for later release.

Regression coverage was expanded so the invariant uses production-representative payloads and verifies that go-live protection reaches automated follow-ups.

The maintainer explicitly accepted the core fix.

## Why the PR is still open

Making the hold work correctly changes behavior for existing installations. Older installations can contain go-live state that historically did not block these follow-ups. Enforcing the corrected rule immediately could therefore retain automated messages for already-operating numbers.

The upstream maintainers added migration and rollout work on top of the original finding and fix, including transition rules and additional invariants. The project classifies the behavior change as requiring the **2.0.0** release, so auto-merge is intentionally disabled.

The later migration/rollout commits are maintainer work and are not presented here as my implementation.

## QA lesson

**A passing test is not evidence of production correctness when the fixture is unrealistic.**

Test data must be traced back to real producers, contracts or observed payloads. In this case, validating the fixture itself exposed the defect that the existing invariant could not see.

---

[← DeskcommCRM overview](overview.md) · [← Portfolio](../../README.md)
