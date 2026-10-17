# DeskcommCRM: Open-Source Contributions

**TL;DR:** DeskcommCRM is an open-source (MIT) self-hosted CRM with AI agents and WhatsApp. I found three production defects, traced each one to its root cause, wrote the fix and added regression tests that fail before the fix and pass after it.
The maintainer reproduced and measured each one. Two are merged into `main`. The third was accepted as is and is waiting for the 2.0.0 release.

Related documents: [PR #1473](pr-1473-ai-agent-publishing.md) · [PR #2324](pr-2324-webhook-retention.md) · [PR #2327](pr-2327-followup-go-live.md)

---

## The project

| | |
|---|---|
| Repository | [melgarafael/DeskcommCRM](https://github.com/melgarafael/DeskcommCRM) |
| What it is | Self-hosted CRM for businesses that sell by chat, with native AI agents and WhatsApp |
| License | MIT |
| Size (October 2026) | About 4,600 GitHub stars, 1,250 forks, more than 100 contributors |
| Stack | TypeScript, Next.js, Supabase (PostgreSQL + PostgREST), background job queue |

## Why this matters for a QA role

This code is not mine. Another team owns the code, the tests and the release decision.

Each PR went through their review process. The maintainer reads the diagnosis, checks the root cause on `main` and runs the tests. Often the maintainer also breaks the source code on purpose to confirm that the new tests catch it. Every review comment lists what was measured and what was not.

So this is independent evidence that my defect reports are correct and that my regression tests protect what they say they protect.

## Contributions

| PR | Defect | Area | My commit | Regression tests | Status |
|---|---|---|---|---|---|
| [#1473](pr-1473-ai-agent-publishing.md) | The "Publish" button of an AI agent never turns on after saving a draft | AI agent editor | 3 files, +143 / −1 | 5 assertions on a new pure function | Merged on 2026-09-23 |
| [#2324](pr-2324-webhook-retention.md) | Webhook log cleanup silently stops with "414 URI Too Long" | Data retention job | 3 files, +128 / −20 | 4 new unit tests | Merged on 2026-10-05 |
| [#2327](pr-2327-followup-go-live.md) | The safety hold for new WhatsApp numbers does not hold automatic follow-ups | AI agent engine (job queue) | 2 files, +104 / −6 | 2 new database invariant cases | Open: accepted as is, waiting for the 2.0.0 release |

## How I worked on each defect

1. **Observe:** I saw the problem on a real production installation and measured it.
2. **Isolate:** I read the code path and found the exact condition that causes it.
3. **Prove:** I wrote a test that fails on the current `main` because of the defect.
4. **Fix:** I changed the smallest piece of code that removes the cause.
5. **Guard the opposite case:** I added a test that shows the fix does not go too far. For example, a real edit must still count as an edit.
6. **Report:** the PR describes the current behavior, the root cause, the fix, the tests, and exactly what I ran and what I could not run.

## What the reviews found

The reviews were not just approvals. The maintainer also added or corrected things:

- **#1473:** the project needs a release-note file for every user-visible change. This rule was not in the contributing guide, so the maintainer added the file in a separate commit.
- **#2324:** CI found a test double in a third test file that my change also affected, and the maintainer updated it. The maintainer also added a test that locks the batch sort order.
- **#2327:** the maintainer corrected one part of my analysis. When the channel was down, the follow-up was re-queued, not sent. The fix itself was accepted as is.

I keep these notes because they are the most useful lessons: search every test double of a function you change, and separate what you measured from what you assumed.

## Techniques used

- Root-cause analysis on real production data
- Regression tests that fail before the fix and pass after it
- Contrast tests that guard against over-fixing
- Database invariant tests on real PostgreSQL (`test:db`, run on PostgreSQL 15 and 17 in CI)
- Boundary analysis (URL size limit, timestamp precision)
- Written defect reports with steps, expected result and actual result

## How I work with AI tools

I use an AI coding assistant (Claude Code) in this work, and two of the PR descriptions say so. I own the diagnosis, the choice of what to test, the verification and the final content of each PR.

[← Back to portfolio](../../README.md)
