# Sanderson Palmeira de Oliveira: QA and Software Testing Portfolio

I am a technology professional and SaaS founder from Brazil with hands-on experience in software quality, AI feature validation, defect investigation, regression testing and production release validation. I have designed, built, tested and shipped two production SaaS products, contributed production bug fixes and regression tests to an established open-source project, and published reproducible defect reports against third-party open-source systems.

I am pursuing a dedicated **Software QA / AI Feature Testing** role in the United States.

The commercial products below have private source code. This public portfolio documents how I test them: strategy, representative test cases, defect investigation, AI behavior validation and release evidence. Customer and patient data is excluded. Repository counts were measured in October 2026.

---

## Evidence at a glance

- **4 third-party open-source defect reports** with reproducible steps and technical evidence; three led to merged fixes, and one was independently reproduced and consolidated into a maintainer root-cause plan.
- **2 upstream open-source fixes merged** into DeskcommCRM after independent maintainer validation.
- **1 additional upstream core fix accepted**, with the PR intentionally waiting for the project's 2.0.0 rollout.
- **2 production SaaS products** tested and shipped: GBP Manager and Agiliza Terapias.
- **7,594 Vitest test cases in 479 files** documented for Agiliza Terapias, plus Playwright browser smoke coverage.
- **251 automated test files** documented for GBP Manager, including Playwright desktop and mobile journeys.
- **537 of 691 fix-prefixed commits (77.7%)** across the two products also changed test coverage in an October 2026 commit-history audit.
- Hands-on evidence of **functional, regression, negative, boundary-value, permission, API, AI-behavior and release testing**.
- Tooling includes **Playwright, Vitest, Supertest, node:test, GitHub Actions and Sentry**.

---

## Third-party defect reporting

### [Open-source bug reports](projects/open-source-bug-reports.md)

These reports map directly to a core QA responsibility: detect a defect, make it reproducible, document expected versus actual behavior, provide enough evidence for engineering to act, and track the result.

| Report | Evidence | Outcome |
|---|---|---|
| [Krayin #2680](https://github.com/krayin/laravel-crm/issues/2680) | Blank optional phone could produce HTTP 500; report included reproduction, code-level cause and a second edge-case variant | Closed as completed; maintainer fix [#2693](https://github.com/krayin/laravel-crm/pull/2693) merged |
| [Krayin #2681](https://github.com/krayin/laravel-crm/issues/2681) | Missing `product_id` bypassed server validation and failed at the database | Contributor fix [#2682](https://github.com/krayin/laravel-crm/pull/2682) with regression tests approved and merged |
| [DeskcommCRM #2326](https://github.com/melgarafael/DeskcommCRM/issues/2326) | SQL reproduction and event-by-event root-cause analysis for automations failing with `service_boundary_stale` | Fix [#2330](https://github.com/melgarafael/DeskcommCRM/pull/2330) merged; contributor publicly credited the reproduction measurements |
| [claude-mem #3906](https://github.com/thedotmack/claude-mem/issues/3906) | Five-day outage diagnosed through logs, health state and database timestamps while health checks stayed green | Independently reproduced; maintainer consolidated root cause and fix sequence into [#3609](https://github.com/thedotmack/claude-mem/issues/3609) |

The Krayin and claude-mem reports are in English; the DeskcommCRM report is in Portuguese. The original issues disclose AI-assisted drafting where applicable.

---

## Regression discipline

A local Git-history audit found that **537 of 691 commits whose subject begins with `fix` (77.7%) also changed test coverage** across GBP Manager and Agiliza Terapias. The exact method, per-product counts and limitations are documented in [Regression-test discipline from commit history](projects/regression-discipline.md).

---

## Open-source QA contributions

### [DeskcommCRM](projects/deskcommcrm/overview.md)

These contributions are the strongest independent evidence in this portfolio because the product, review process and merge decision are controlled by an external open-source maintainer.

| Contribution | QA evidence | Status |
|---|---|---|
| [PR #1473 — AI agent publishing](projects/deskcommcrm/pr-1473-ai-agent-publishing.md) | Reproduced a production UI-state defect, identified semantic-vs-serialized JSON comparison as the cause, added regression coverage | **Merged upstream** |
| [PR #2324 — webhook retention](projects/deskcommcrm/pr-2324-webhook-retention.md) | Traced a silent retention failure to oversized PostgREST URLs, redesigned the boundary and added regression tests | **Merged upstream** |
| [PR #2327 — go-live follow-up hold](projects/deskcommcrm/pr-2327-followup-go-live.md) | Found that a passing invariant used a payload shape not emitted by real producers; added production-representative coverage | **Core fix accepted; open for 2.0.0 rollout** |

Direct upstream evidence: [#1473](https://github.com/melgarafael/DeskcommCRM/pull/1473) · [#2324](https://github.com/melgarafael/DeskcommCRM/pull/2324) · [#2327](https://github.com/melgarafael/DeskcommCRM/pull/2327)

---

## Production projects

### [GBP Manager](projects/gbp-manager/overview.md)

A multi-tenant SaaS for Google Business Profiles: posts, review replies, local-rank insights and AI content. I test by risk. Client-data isolation, billing, Google integration, AI output and text quality come first.

| Document | What you will find |
|---|---|
| [Overview](projects/gbp-manager/overview.md) | Product, users, tech stack, my role, timeline |
| [Architecture](projects/gbp-manager/architecture.md) | System diagram, AI request path, CI/CD and deploy |
| [QA strategy](projects/gbp-manager/qa-strategy.md) | Risk-based testing, test pyramid, CI gates, entry and exit criteria, severity scale, Sentry monitoring |
| [Test cases](projects/gbp-manager/test-cases.md) | 25 test cases (functional, negative, mobile, permission, AI output) with traceability |
| [Bug case study](projects/gbp-manager/bug-case-study.md) | Two real production defects in standard defect-report format |
| [Release validation](projects/gbp-manager/release-validation.md) | Release flow, go/no-go checklist, evidence records, sample sign-off |

### [Agiliza Terapias](projects/agiliza-terapias/overview.md)

A practice-management product for therapy clinics with scheduling, patient records, billing, a WhatsApp AI assistant and AI clinical notes. AI behavior is surrounded by deterministic, tested controls before a message can be sent.

| Document | What you will find |
|---|---|
| [Overview](projects/agiliza-terapias/overview.md) | Product, users, tech stack, my role, timeline |
| [Architecture](projects/agiliza-terapias/architecture.md) | System diagram and how an AI request flows through the guardrails |
| [QA strategy](projects/agiliza-terapias/qa-strategy.md) | Risk-based testing, test levels, test data policy, CI/CD gates, severity scale |
| [Test cases](projects/agiliza-terapias/test-cases.md) | 25 test cases, AI-veto decision table, boundary values and traceability |
| [AI feature testing](projects/agiliza-terapias/ai-feature-testing.md) | AI risks, guardrail chain, guardrail-to-test mapping, behavior scenarios and known gaps |
| [Release validation](projects/agiliza-terapias/release-validation.md) | Release flow, migration safety, release checklist and sample pre-release note |

### Project numbers

| | GBP Manager | Agiliza Terapias |
|---|---|---|
| Commits | 853 | 1,280 |
| Automated tests | 251 test files, including Playwright journeys on Desktop Chrome and a Pixel 7 mobile profile | 479 test files with 7,594 test cases (Vitest + Supertest) |
| Release records | 63 homologation sign-offs, 46 pre-release manifests, 40 evidence records | 301 pre-release notes |
| AI quality | Unsafe AI output is blocked and replaced by a safe fallback | Every assistant message passes a tested guardrail chain before it is sent |
| CI gates | Typecheck, critical-flow guard, critical test suite, build | Lint, build, tests, database migration check |

---

## QA skills and evidence

| Skill | Evidence |
|---|---|
| Production defect investigation and root-cause analysis | [Third-party defect reports](projects/open-source-bug-reports.md), [DeskcommCRM contributions](projects/deskcommcrm/overview.md), [GBP bug case study](projects/gbp-manager/bug-case-study.md) |
| Functional testing of AI features | [Agiliza AI feature testing](projects/agiliza-terapias/ai-feature-testing.md), AI-output cases in [GBP test cases](projects/gbp-manager/test-cases.md) |
| Test cases across scenarios and conditions, including mobile | [GBP test cases](projects/gbp-manager/test-cases.md), [Agiliza test cases](projects/agiliza-terapias/test-cases.md) |
| Regression testing | All three [DeskcommCRM cases](projects/deskcommcrm/overview.md), plus both product QA strategies |
| Test-design techniques | Equivalence partitioning, boundary values, decision tables, state transitions and error guessing in the product test-case documents |
| Defect reporting with reproducible steps | [Third-party defect reports](projects/open-source-bug-reports.md) |\n| Test documentation and traceability | [GBP QA strategy](projects/gbp-manager/qa-strategy.md), [Agiliza QA strategy](projects/agiliza-terapias/qa-strategy.md) |
| Release validation across the SDLC | [GBP release validation](projects/gbp-manager/release-validation.md), [Agiliza release validation](projects/agiliza-terapias/release-validation.md) |
| Test automation and CI | Playwright, node:test, Vitest, Supertest and GitHub Actions |\n| Regression discipline | [537 of 691 fix-prefixed commits changed tests](projects/regression-discipline.md), plus independently inspectable upstream regression cases |
| Production monitoring | Sentry and health/release evidence documented in both product QA strategies |
| Test-fixture validation | [DeskcommCRM PR #2327](projects/deskcommcrm/pr-2327-followup-go-live.md) |

---

## Tools

- **Testing:** Playwright, Vitest, Supertest, node:test, GitHub Actions, Sentry
- **Development / systems:** TypeScript, React, Node.js, Express, PostgreSQL (Supabase), Docker, Git
- **AI:** OpenAI and Anthropic APIs, structured output (JSON Schema), tool calls, deterministic guardrails and provider fallback

### AI-assisted development

I use AI coding tools as part of my development workflow. I remain responsible for requirements, production observations, defect definition, test strategy, acceptance criteria, review and release decisions. Implementation and test drafting can be AI-assisted; I validate the result against the real system behavior before it is accepted or submitted upstream.

---

## Repository structure

```text
README.md
projects/
    deskcommcrm/
        overview.md
        pr-1473-ai-agent-publishing.md
        pr-2324-webhook-retention.md
        pr-2327-followup-go-live.md
    gbp-manager/
        overview.md
        architecture.md
        qa-strategy.md
        test-cases.md
        bug-case-study.md
        release-validation.md
    agiliza-terapias/
        overview.md
        architecture.md
        qa-strategy.md
        test-cases.md
        ai-feature-testing.md
        release-validation.md
assets/
    screenshots/
```

---

## Contact

GitHub: [github.com/Sandersono](https://github.com/Sandersono)  
LinkedIn: [linkedin.com/in/sanderson-oliveira](https://www.linkedin.com/in/sanderson-oliveira/)
