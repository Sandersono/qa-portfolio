# Sanderson Palmeira de Oliveira: QA and Software Testing Portfolio

I am a full-stack developer and solo founder from Brazil. I built, tested and shipped two production SaaS products, both with AI features, and I am looking for a Software Tester / QA role in the US.

Both products are commercial and their source code is private. This portfolio shows how I test them: strategy, test cases, defect reports and release validation. Customer data is removed, and every number below was counted in the repositories in October 2026.

---

## At a glance

| | GBP Manager | Agiliza Terapias |
|---|---|---|
| What it is | SaaS to manage Google Business Profiles for local businesses and agencies | Practice-management app for therapy clinics, with an AI WhatsApp assistant and AI clinical notes |
| Commits | 853 | 1,280 |
| Automated tests | 251 test files, including Playwright journeys on Desktop Chrome and a Pixel 7 mobile profile | 479 test files with 7,594 test cases (Vitest + Supertest) |
| Release records | 63 homologation sign-offs, 46 pre-release manifests, 40 evidence records | 301 pre-release notes |
| AI quality | Unsafe AI output is blocked and replaced by a safe fallback | Every assistant message passes a tested guardrail chain before it is sent |
| CI gates | Typecheck, critical-flow guard, critical test suite, build | Lint, build, tests, database migration check |

---

## Projects

### [GBP Manager](projects/gbp-manager/overview.md)

A multi-tenant SaaS for Google Business Profiles: posts, review replies, local rank insights and AI content. I test by risk. Client data isolation, billing, the Google integration, AI output and text quality come first.

| Document | What you will find |
|---|---|
| [Overview](projects/gbp-manager/overview.md) | Product, users, tech stack, my role, timeline |
| [Architecture](projects/gbp-manager/architecture.md) | System diagram, AI request path, CI/CD and deploy |
| [QA strategy](projects/gbp-manager/qa-strategy.md) | Risk-based testing, test pyramid, CI gates, entry and exit criteria, severity scale, Sentry monitoring |
| [Test cases](projects/gbp-manager/test-cases.md) | 25 test cases (functional, negative, mobile, permission, AI output) with traceability |
| [Bug case study](projects/gbp-manager/bug-case-study.md) | Two real production defects in standard defect report format |
| [Release validation](projects/gbp-manager/release-validation.md) | Release flow, go/no-go checklist, evidence records, sample sign-off |

### [Agiliza Terapias](projects/agiliza-terapias/overview.md)

A mental-health practice app: scheduling, patient records, billing, a WhatsApp AI assistant and AI clinical notes. A wrong AI message can hurt a real person, so the AI runs inside fixed, tested guardrails.

| Document | What you will find |
|---|---|
| [Overview](projects/agiliza-terapias/overview.md) | Product, users, tech stack, my role, timeline |
| [Architecture](projects/agiliza-terapias/architecture.md) | System diagram, how an AI request flows through the guardrails |
| [QA strategy](projects/agiliza-terapias/qa-strategy.md) | Risk-based testing, test levels, test data policy, CI/CD gates, severity scale |
| [Test cases](projects/agiliza-terapias/test-cases.md) | 25 test cases, a decision table for the AI veto chain, boundary values, traceability |
| [AI feature testing](projects/agiliza-terapias/ai-feature-testing.md) | AI risks, guardrail chain, guardrail-to-test mapping, AI behavior scenarios, known gaps |
| [Release validation](projects/agiliza-terapias/release-validation.md) | Release flow, migration safety, release checklist, sample pre-release note |

---

## QA skills and where to see them

| Skill | Evidence |
|---|---|
| Functional testing of AI features (accuracy, consistency, intended behavior) | [AI feature testing](projects/agiliza-terapias/ai-feature-testing.md), AI output cases in [GBP test cases](projects/gbp-manager/test-cases.md) |
| Test cases across scenarios and conditions, including mobile | [GBP test cases](projects/gbp-manager/test-cases.md) (Pixel 7 profile), [Agiliza test cases](projects/agiliza-terapias/test-cases.md) |
| Defect reports with reproducible steps | [Bug case study](projects/gbp-manager/bug-case-study.md) |
| Test design techniques (equivalence partitioning, boundary values, decision tables, state transition) | [GBP test cases](projects/gbp-manager/test-cases.md), [Agiliza test cases](projects/agiliza-terapias/test-cases.md) |
| Test documentation that follows a standard (scope, risks, entry and exit criteria, severity, traceability) | [GBP QA strategy](projects/gbp-manager/qa-strategy.md), [Agiliza QA strategy](projects/agiliza-terapias/qa-strategy.md) |
| Release validation across the software life cycle | [GBP release validation](projects/gbp-manager/release-validation.md), [Agiliza release validation](projects/agiliza-terapias/release-validation.md) |
| Test automation and CI | Playwright, node:test, Vitest and Supertest in both QA strategies |
| Production monitoring | Sentry sections in both QA strategies |

---

## Tools

- **Testing:** Playwright, Vitest, Supertest, node:test, GitHub Actions, Sentry
- **Development:** TypeScript, React, Node.js, Express, PostgreSQL (Supabase), Docker, Git
- **AI:** OpenAI and Anthropic APIs, structured output (JSON schema), guardrails and fallbacks

---

## Repository structure

```
README.md
projects/
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
