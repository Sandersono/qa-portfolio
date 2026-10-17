# GBP Manager: Product Overview

**TL;DR:** GBP Manager is a multi-tenant SaaS that helps local businesses and agencies manage their Google Business Profiles.
I built it alone. I own product, development, QA and release management.
The main branch has 251 automated test files, 63 homologation (sign-off) scripts and a gated release flow.

Related documents: [Architecture](architecture.md) · [QA strategy](qa-strategy.md) · [Test cases](test-cases.md) · [Bug case study](bug-case-study.md) · [Release validation](release-validation.md)

---

## What the product does

GBP Manager connects to the Google Business Profile API. A business (or the agency that serves it) manages its public Google profile from one panel:

- posts, reviews and replies, opening hours, services, photos and local search rank;
- AI help: review replies, post text, profile keywords and a local rank insight;
- monthly reports by e-mail and PDF.

A separate module, **Prospecção** (prospecting), audits a business profile and creates a print-safe A4 report for sales conversations.

- Product: `app.gbpmanager.com.br` (login required)
- Public site: `gbpmanager.com.br`

## Who uses it

| User type | What they do in the product |
|---|---|
| Local business owner (client user) | Sees the dashboard, reports and local map. Changes basic automation settings. Pays the subscription. Opens support tickets. |
| Agency operator / admin | Manages several business locations: posts, replies, services, media, hours. |
| Superadmin (operator of the platform) | Manages clients, finance, operations (jobs and failures), automations and access. |

The UI is bilingual: Brazilian Portuguese (main language) and English, through a translation helper. My tests check both languages.

## Key features

| Area | What it does |
|---|---|
| Signup and onboarding | Connect a Google account in a popup, choose locations, pay through the Asaas checkout. Guided activation steps for new clients. |
| Posts | Scheduled posts, editorial calendar, events and products published as posts, automatic posts written by AI. |
| Reviews | Review sync, AI replies, negative review alerts, sentiment trend, review requests. |
| Profile and hours | Special hours, holidays, and profile protection ("blindagem") that finds and repairs unwanted changes to protected hours. |
| Services and SEO | Services and descriptions, plus a keyword ("semantic") profile that feeds posts and replies. |
| Local map ("Mapa Local") | Monthly geo-grid rank scan, competitors, and an AI insight with 3 actions. A deterministic fallback replaces unsafe or invalid AI output. |
| Media | Auto-Gallery publishes photos from Google Drive folders, with a dry-run planner. Video publishing sits behind a rollout flag. |
| Reports | Monthly executive report (e-mail and PDF) and KPI views. |
| Help and support | Help center with an assistant, and manual tickets to human support. |
| Billing | Asaas subscriptions, webhooks and payment reminders. When payment is pending, the client is redirected to the payments screen. |
| Operations | Background jobs with run history, a dead-letter list and failure alerts. |
| Prospecção | Profile audit with an AI narrative, a safety filter for sales language, and an A4 report. |

## Tech stack

| Layer | Technology |
|---|---|
| Language | TypeScript 5.8 |
| Front end | React 19, Vite 6, Tailwind CSS 4, React Router 7, Recharts |
| Back end | Node.js 20, Express 4 |
| Database | PostgreSQL (Supabase), migrations with node-pg-migrate |
| Integrations | Google Business Profile API and Google OAuth (google-auth-library), Google Drive, Asaas billing, OpenAI Responses API, Google Places API (prospecting module) |
| Testing | node:test + assert (run with tsx), Playwright (Desktop Chrome and Pixel 7 profiles), custom Node gate scripts |
| CI/CD | GitHub Actions, git pre-push hook, Docker images, deploy script on a VPS |
| Monitoring | Sentry (back end and browser), health endpoints `/api/livez` and `/api/readyz` |

## My role

I am the solo founder. I do all of these jobs:

- **Product:** backlog with priorities P0 to P2, feature specs, acceptance criteria.
- **Development:** front end, back end, integrations, database.
- **QA:** test strategy, automated tests, homologation scripts, exploratory checks with real providers, defect analysis.
- **Release management:** versions, release notes, evidence records, go/no-go decisions, deploy, production monitoring.

## Project timeline (from git)

| Fact | Value | How I counted |
|---|---|---|
| First commit | February 2026 | `git log --reverse` on main |
| Commits on main | 853 | `git rev-list --count main` (October 2026) |
| Merged pull requests | 188 commits on main reference a squash-merged PR (#1 to #196) | commit subjects that end in `(#N)` |
| Version tags | 62 (v0.9.1 to v0.9.64) | `git tag` |
| Current version | 0.9.64 in `package.json` (tag v0.9.64, May 2026) | `package.json` |
| Latest homologation script | v0.9.81 (September 2026) | `docs/homologacao-v0.9.*.md` |
| Automated test files | 251 (`*.test.ts`, `*.test.tsx`, `*.spec.ts`) | `git ls-files` on main, no `node_modules`, no worktrees |

Note: after v0.9.64 I kept writing one homologation script per change (v0.9.65 to v0.9.81), but I did not bump the package version. I explain this gap in [Release validation](release-validation.md#known-gaps-and-next-steps).

### QA milestones

| When | What I added |
|---|---|
| Feb 2026 | CI workflow, PR flow rules and the Main Guard workflow (no direct push to main) |
| Apr 2026 | Local release gate (`release:check`) |
| May 2026 | Critical-flow guard (`smoke:critical`), Playwright journeys on desktop and Pixel 7, one evidence record per release |
| Jul 2026 | PR-only flow in daily use: 187 of the 200 commits since July came from pull requests |
| Aug 2026 | Sentry on the back end, then in the browser |
| Sep 2026 | Text-quality scan, accent guard tests, fix for AI input cut in the middle of an emoji |

## What this project shows for a QA role

| QA duty | Where you can see it |
|---|---|
| Functional testing of AI features | AI insight tests: unsafe phrase blocked, invalid JSON, AI turned off, all go to a safe fallback. Report safety filter. Prompt fingerprint for traceability. See [Test cases](test-cases.md). |
| Test cases across scenarios, languages and conditions | Desktop and Pixel 7 runs, Portuguese and English locators, accented and emoji input, billing states. |
| Reproducible defect reports | Two real defects with steps, expected vs actual, root cause and verification. See [Bug case study](bug-case-study.md). |
| Test documentation that follows a standard | Test strategy, go/no-go checklist, 63 homologation scripts, 40 release evidence records. |
| Work across all SDLC phases | Specs, then tests in the PR, CI gates, homologation, go/no-go, deploy and Sentry monitoring. |
| Regression control | A critical suite runs in CI on every PR. A guard script fails the build if a critical test is deleted. |

---

[← Back to portfolio](../../README.md)
