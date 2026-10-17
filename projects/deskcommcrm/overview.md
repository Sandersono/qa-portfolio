# DeskcommCRM: Open-Source QA Contributions

**TL;DR:** I investigated production defects in DeskcommCRM, reproduced the failures, identified their root causes, proposed fixes and added regression coverage. Two pull requests were merged upstream after maintainer validation. A third core fix was accepted by the maintainer and is waiting for the project's 2.0.0 rollout because it changes existing installation behavior.

This project is useful evidence because the product and review process are not controlled by me. The upstream maintainer independently checked the diagnoses, ran additional tests and documented the results publicly.

## The project

| | |
|---|---|
| Repository | [melgarafael/DeskcommCRM](https://github.com/melgarafael/DeskcommCRM) |
| What it is | Self-hosted CRM for businesses that sell by chat, with native AI agents and WhatsApp |
| License | MIT |
| Size (October 2026) | About 4,600 GitHub stars, 1,250 forks, more than 100 contributors |
| Stack | TypeScript, Next.js, Supabase (PostgreSQL + PostgREST), background job queue |

## Contributions

| PR | Problem | QA / engineering evidence | My commit | Status |
|---|---|---|---|---|
| [#1473](pr-1473-ai-agent-publishing.md) | AI-agent changes could remain impossible to publish after saving a draft | Production reproduction, semantic-state analysis, regression tests, independent maintainer validation | 3 files, +143 / −1 | **Merged** (2026-09-23) |
| [#2324](pr-2324-webhook-retention.md) | Webhook-retention pruning could silently stop because a PostgREST URL became too large | Operational failure analysis, boundary/root-cause reasoning, regression tests, maintainer challenge testing | 3 files, +128 / −20 | **Merged** (2026-10-05) |
| [#2327](pr-2327-followup-go-live.md) | A passing invariant used a payload shape that real producers did not emit, so go-live protection missed automated follow-ups | Test-fixture validation, producer inspection, invariant regression coverage, release-transition analysis | 2 files, +104 / −6 | **Core fix accepted; PR open for 2.0.0 rollout** |

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

## What these contributions demonstrate

- Reproducing defects from real production behavior rather than only from specifications.
- Root-cause analysis across UI state, JSON serialization, PostgREST, database behavior and asynchronous job payloads.
- Writing regression tests that distinguish the failure from nearby valid behavior.
- Checking whether an existing test fixture represents real production data.
- Working inside another project's review, CI and release process.
- Responding to maintainer corrections without hiding uncertainty or overstating the scope of a fix.

## AI-assisted workflow

I use AI coding tools as part of my development workflow. The production observations, problem definition, validation questions, acceptance criteria and final verification remain human-led. Some implementation work and test drafting can be AI-assisted; I review the result against the real failure and the repository's behavior before submitting it.

This distinction matters in the case studies below: the evidence is not that a tool generated code, but that the reported production failures were reproducible, the proposed tests could fail for the intended reasons, and upstream maintainers independently validated the contributions.

---

[← Back to portfolio](../../README.md)
