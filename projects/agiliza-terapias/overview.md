# Agiliza Terapias: Product Overview

**TL;DR:** Agiliza Terapias is a practice-management web app for therapy clinics and solo therapists in Brazil (scheduling, patient records, AI clinical notes, a WhatsApp AI assistant, billing).
I built it alone. I own the product, the code, the QA and the releases.
For QA, this project has 7,594 automated test cases, a tested safety guardrail chain for the AI assistant, and 301 pre-release notes.

Related documents: [Architecture](architecture.md) · [QA strategy](qa-strategy.md) · [Test cases](test-cases.md) · [AI feature testing](ai-feature-testing.md) · [Release validation](release-validation.md)

---

## What the product does

Agiliza Terapias helps a therapy practice run its daily work in one place:

- The team books, reschedules and confirms sessions.
- Therapists keep patient records and clinical notes.
- AI turns a session recording into a draft clinical note. The therapist reviews it.
- An AI assistant answers patients on the clinic's WhatsApp. It can offer free times, book and reschedule. It hands the conversation to a human when it should not answer.
- The clinic sends reminders, charges patients and tracks payments.

Website: `agilizaterapias.com.br` (app: `app.agilizaterapias.com.br`). The user interface is in Portuguese.

## Who uses it

| User type | What they do in the product |
|---|---|
| Therapists (psychology, neuropsychology, child therapy, couples therapy) | Record sessions, review AI notes, manage their schedule |
| Clinic administrators | Configure the clinic, team, billing, automations and the AI assistant |
| Receptionists / secretaries | Answer WhatsApp, book and reschedule, register payments. No access to clinical notes |
| Patients and new contacts | Talk to the clinic on WhatsApp, receive reminders, book on a public page |
| Platform operator (me) | Internal admin console: health checks, AI usage, release notes |

## Key features

| Area | What it does |
|---|---|
| Scheduling | Day, week and month views, recurring sessions, blocked times, no-shows, public online booking page, Google Calendar sync |
| Patient records | Patient file, consent, clinical notes with audit history, trash with restore, PDF export |
| AI clinical notes | Speech-to-text of the session audio, then a structured draft note in the clinic's note format. Risk highlight at the top of the note. Reprocess without a new recording |
| Desktop recorder | Windows app (Tauri) that records the session, encrypts the file on the computer, uploads it in parts and resumes after a network failure |
| WhatsApp inbox ("Atendimento") | Shared inbox for the team: internal notes, quick replies, labels, AI reply suggestion ("Sugerir resposta" = Suggest reply) |
| WhatsApp AI assistant | Answers patients, lists free times, books and reschedules (it never cancels), uses the clinic's knowledge base, hands off to staff. Every message passes a guardrail chain before it is sent |
| Automations | Visual flow builder for reminders, confirmations and lead follow-up. A flow does nothing until it is published |
| Confirmations | Reads the patient's reply to a reminder. Clear replies use fixed rules; unclear replies go to an AI classifier |
| Billing ("Financeiro") | Charges, payments, bank slips (boletos), an optional "pay before confirm" rule, financial reports. Asaas integration |
| Roles and permissions | Admin, professional and secretary see different menus and data |
| Privacy (LGPD) | Patient data export, erasure requests, logs and error reports without personal data |

## Tech stack

| Layer | Technology |
|---|---|
| Frontend | React 19, Vite 6, Tailwind CSS 4, TypeScript 5.8 |
| Backend | Node.js 22, Express 4, Zod 4 (input validation) |
| Database and auth | Supabase (Postgres, Auth, Storage) |
| AI | OpenAI (chat models, speech-to-text, embeddings); Anthropic (backup provider for the WhatsApp assistant) |
| Messaging | Self-hosted WhatsApp HTTP gateway (Docker container), SMTP email |
| Integrations | Asaas (billing), Google Calendar, a bank API for boletos |
| Desktop app | Tauri 2 + React (Windows session recorder) |
| Infrastructure | Docker Compose, Caddy (TLS reverse proxy), one VPS, GitHub Container Registry |
| CI/CD | GitHub Actions |
| Monitoring | Sentry (browser and server) |
| Testing and quality | Vitest 3, Supertest 7, Playwright 1.52, TypeScript compiler check, ESLint 9 (typescript-eslint, react-hooks) |

## My role

I am the solo founder. I do every job on this product:

- **Product:** I write the requirements and specs, and I decide the rules (for example, "the assistant goes silent and calls a human when it does not know").
- **Development:** full stack (React, Express, Postgres migrations, the desktop app).
- **QA:** I design the tests, write the automated tests, run the manual real-world checks and log defects with steps.
- **Release management:** versioning, release notes, migration checks, deploy, rollback plan.
- **Operations:** health checks and Sentry monitoring after each release.

## Timeline facts (from git and the repo)

| Fact | Value | How I counted it |
|---|---|---|
| First commit | February 2026 (2026-02-21) | `git log --reverse` |
| Last commit counted | 2026-10-15 | `git log -1` |
| Commits on main | 1,280 | `git rev-list --count main` on 2026-10-16 |
| Current version | 0.4.0 | `version` in `package.json` |
| Pre-release notes | 301 (v0.1.1 to v0.4.0) | files `docs/releases/pre-release-*.md` |
| Automated test files | 479 Vitest files + 1 Playwright file | files `tests/backend/**/*.test.{ts,tsx}` and `tests/e2e/*.spec.ts` |
| Automated test cases | 7,594 Vitest + 4 Playwright | `npx vitest list --json`; `test(` calls in the Playwright spec |
| Database migrations | 145 SQL files | files in `supabase/migrations/` |

## What this project shows for an AI-feature QA role

| Job need | Where I show it |
|---|---|
| Functional testing of AI features: accuracy, consistency, intended behavior | Guardrail chain tests, strict JSON schema tests for clinical notes, low-temperature and fixed-rule checks. See [AI feature testing](ai-feature-testing.md) |
| Test cases across scenarios and conditions | Decision table for the guardrail chain, boundary values for clinic hours, daily AI budget and usage limits, time zone cases. See [Test cases](test-cases.md) |
| Reproducible defect reports | Each release note lists the defect, the fix, the tests added, known limits and manual check steps. See [Release validation](release-validation.md) |
| Test documentation that follows a standard | Risk-based test strategy, entry and exit criteria, severity scale, traceability from test case to test file. See [QA strategy](qa-strategy.md) |
| Work in all SDLC phases | Spec, code, automated tests, CI gates, migration check, release note, deploy smoke test, Sentry monitoring |
| Honesty about gaps | No AI evaluation dataset yet, few end-to-end browser tests. I list these as next steps |

---

[← Back to portfolio](../../README.md)
