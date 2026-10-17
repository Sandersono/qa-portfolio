# Agiliza Terapias: Architecture

**TL;DR:** One Express API serves a React single-page app and talks to Supabase, a WhatsApp gateway, AI providers, billing and calendar services. It runs with Docker Compose behind Caddy on one VPS, and GitHub Actions builds and deploys it.
For QA, the key part is the AI path: every message from the WhatsApp assistant passes risk checks, a daily budget check and a fixed veto chain before it is sent.

Related documents: [Overview](overview.md) · [QA strategy](qa-strategy.md) · [Test cases](test-cases.md) · [AI feature testing](ai-feature-testing.md) · [Release validation](release-validation.md)

---

## 1. System context

```mermaid
flowchart LR
  subgraph Clients
    Staff["Clinic staff<br/>browser"]
    Visitor["New patient<br/>public booking page"]
    Recorder["Desktop recorder<br/>Tauri app on Windows"]
    Patient["Patient<br/>WhatsApp"]
  end

  subgraph VPS["VPS with Docker Compose"]
    Caddy["Caddy<br/>TLS reverse proxy"]
    App["App container<br/>Express API + React SPA build"]
    Gateway["WhatsApp gateway<br/>container"]
  end

  subgraph Supabase["Supabase"]
    DB[("Postgres")]
    Auth["Auth"]
    Storage["Storage<br/>recordings"]
  end

  subgraph AI["AI providers"]
    OpenAI["OpenAI<br/>chat, speech-to-text, embeddings"]
    Anthropic["Anthropic<br/>backup for the assistant"]
  end

  Staff --> Caddy
  Visitor --> Caddy
  Recorder --> Caddy
  Caddy --> App
  Patient <--> Gateway
  Gateway <-->|webhooks and sends| App
  App --> DB
  App --> Auth
  App --> Storage
  App --> OpenAI
  App -.->|retryable errors only| Anthropic
  App --> GCal["Google Calendar"]
  App --> Billing["Asaas and bank API<br/>billing"]
  App --> SMTP["SMTP<br/>email alerts"]
  App --> Sentry["Sentry"]
  Staff -.-> Sentry
```

| Component | Role | Notes for testers |
|---|---|---|
| React SPA | Staff screens and the public booking page | Render tests check screens as HTML (no browser needed) |
| Express API | All business rules, webhooks, AI calls, background workers | Most automated tests run here, with a mocked database and mocked providers |
| Supabase | Postgres data, user sign-in, file storage for recordings | Schema changes only through versioned migration files |
| WhatsApp gateway | Receives patient messages (webhook) and sends replies | Runs as its own container next to the app |
| OpenAI | Chat models (assistant, notes, classifiers), speech-to-text, embeddings for the knowledge base | Never called for real in automated tests; `fetch` is stubbed |
| Anthropic | Backup chat provider for the WhatsApp assistant only | Used only after a retryable failure of the primary provider |
| Google Calendar, Asaas, bank API | Calendar sync and billing | Enabled step by step, each with a manual smoke test |
| SMTP | Email channel for risk alerts and notices | Second channel next to WhatsApp |
| Desktop recorder | Records the session, encrypts it locally, uploads it in parts | Hardware tests (real microphone) are manual |
| Sentry | Error monitoring in browser and server | Server events are cleaned of personal data before sending |

## 2. Build and deploy

```mermaid
flowchart LR
  PR["Pull request<br/>or manual run"] --> Lint["Lint<br/>tsc + ESLint"]
  Lint --> Build["Build<br/>Vite"]
  Build --> Tests["Tests<br/>Vitest + Supertest"]
  Tests --> DbCheck["Migration lint<br/>db:check"]
  DbCheck -->|manual run on main| Image["Build and push<br/>Docker image"]
  Image --> Migrate["Apply pending<br/>migrations"]
  Migrate --> Deploy["Deploy over SSH<br/>docker compose up"]
  Deploy --> Smoke["Smoke check<br/>health and ready endpoints"]
  Smoke --> Monitor["Sentry<br/>monitoring"]
```

- One workflow file with four jobs: `ci`, `docker_image`, `db_migrate`, `deploy_vps`. Each job needs the previous one to pass.
- The `ci` job runs on every pull request. The deploy jobs run only on a manual run on `main`. Details: [Release validation](release-validation.md).

## 3. How an AI request flows through the guardrails

This is the path of one patient message to the WhatsApp assistant.

```mermaid
sequenceDiagram
  participant P as Patient
  participant G as WhatsApp gateway
  participant API as Express API
  participant LLM as AI provider
  participant S as Clinic staff
  P->>G: Message
  G->>API: Webhook
  API->>API: Save message, wait 8 s for more messages
  API->>API: Risk and handoff check on the incoming text
  alt Risk signal, asks for a human, or clinical topic
    API->>S: Alert and open a case
    Note over API: Assistant stays silent, conversation paused
  else Normal message
    API->>API: Entry gates (human took over, opt-out, service hours)
    API->>API: Daily AI budget gate
    alt Budget cap reached
      API->>S: Hand off with the reason
    else Below the cap
      API->>LLM: Instructions, knowledge, memory, tools
      LLM-->>API: Draft reply or tool call
      API->>API: Veto chain on the draft
      alt Draft vetoed
        API->>LLM: Veto reason, model writes again
        Note over API: After 2 vetoes in a row, hand off to staff
      else Draft approved
        API->>G: Send reply in short messages
        G->>P: Reply
      end
    end
  end
```

Step by step:

1. **Receive.** The gateway sends a webhook. The API saves the message in the inbox first. A server restart can lose a pending automatic reply, but not the message.
2. **Group.** The assistant waits 8 seconds of silence, so "hi / I want to book / for Thursday" gets one reply.
3. **Preconditions.** The assistant runs only if the clinic turned it on and set a phone number for risk alerts. It uses a patient's data only when the WhatsApp number matches the patient's record exactly.
4. **Risk and handoff check (before any AI call).** A fixed detector reads the incoming text. A risk signal means: the assistant does not reply, staff get an urgent alert, and the conversation is paused. A request for a human or a clinical topic (medication, reports) also goes to staff. This runs before the model, so it still works when the AI provider is down or the budget is used up.
5. **Entry gates.** If a staff member took over the conversation and the pause has not ended, the assistant does nothing and spends nothing.
6. **Budget gate.** The daily AI spend of the clinic must be below its cap. If not, there is no AI call and staff get the conversation with the reason.
7. **Model call.** The prompt has the clinic's instructions, matching knowledge base passages, the conversation memory and the tools (list free times, book, reschedule, my sessions). When text written by the person goes into the instructions, it is quoted and marked as data, not as an instruction. Output tokens per turn and tool rounds per turn (3) are limited.
8. **Provider fallback.** If OpenAI fails with a retryable error (network, timeout, 5xx, 429, 401/402/403), the same request goes to Anthropic with the same temperature and tools. A 400 error does not go to the backup.
9. **Veto chain.** Six gates check the draft text, in a fixed order: opt-out, human took over, service hours, forbidden promise, promise to call back without a case, self-identification in the first message. A gate never rewrites text. It sends the reason back to the model, and the model writes again. Two vetoes in a row hand the conversation to staff.
10. **Send and log.** The reply goes out in short messages. Each turn is logged on the "AI Executions" screen (replied, handed off, did not reply, failed), and each AI call writes a usage row with real token counts.

Full guardrail details and tests: [AI feature testing](ai-feature-testing.md).

## 4. AI clinical note flow

```mermaid
flowchart TD
  A["Session recording<br/>desktop app or browser"] --> B{"Recording consent<br/>granted?"}
  B -->|No| X["Blocked<br/>no recording, no AI call"]
  B -->|Yes| C["Upload in parts<br/>encrypted by the desktop app"]
  C --> D["Speech-to-text<br/>parts of up to 8 minutes"]
  D --> E{"Speech found?"}
  E -->|No| Y["No note is generated"]
  E -->|Yes| F["Generate note<br/>strict JSON schema, temperature 0.2"]
  F --> G{"Model accepts<br/>the strict schema?"}
  G -->|No| H["Fallback: plain JSON mode"]
  G -->|Yes| I["Parse and check fields"]
  H --> I
  I --> J["Draft note for therapist review"]
```

- The note is always a draft. The therapist reviews and finalizes it.
- The clinical note has no backup provider. If OpenAI fails, the user sees an error and tries again. A retry of the same session is not billed twice.

## 5. Code layout (for developers)

| Path | Content |
|---|---|
| `src/` | React SPA screens and components |
| `src/lib/` | Shared pure logic (guardrail chain, handoff rules, budget, clinic hours). Easy to unit test |
| `src/backend/` | Controllers and services (AI, agent, inbox, billing, automations) |
| `server.ts` | Express app setup, middleware order, Sentry init |
| `tests/backend/` | 479 Vitest test files (unit, API with Supertest, render, migration) |
| `tests/e2e/` | Playwright smoke spec |
| `supabase/migrations/` | 145 versioned SQL migrations |
| `ops/` | Migration runner, preflight checks, rollback script and rollback SQL |
| `apps/` | Desktop recorder (Tauri) |

---

[← Back to portfolio](../../README.md)
