# Agiliza Terapias: Representative Test Cases

**TL;DR:** These 25 test cases come from real tests in the product. They cover scheduling, patient records, billing, the WhatsApp inbox, automations, AI features and permissions.
Each case lists the design technique, the test type and how it runs. A traceability table at the end links each case to its test file.

Related documents: [Overview](overview.md) · [Architecture](architecture.md) · [QA strategy](qa-strategy.md) · [AI feature testing](ai-feature-testing.md) · [Release validation](release-validation.md)

---

## How to read this

- The product UI is in Portuguese. I quote the Portuguese text first, then the English translation in parentheses.
- "Assistant" means the WhatsApp AI assistant. "Gate" means one step of the assistant's veto chain.
- All data in these tests is synthetic.
- Execution values: Automated – Vitest (unit or service test), Automated – Supertest (HTTP API test inside Vitest), Automated – Playwright (browser), Manual.

## Test case table

| ID | Title | Preconditions | Steps | Expected result | Technique | Type | Execution |
|---|---|---|---|---|---|---|---|
| AGT-TC-01 | Two bookings race for the same slot | A professional with a free slot. Booking made by the assistant | 1. The assistant reads the slot as free.<br>2. Another booking takes the slot before the save.<br>3. The assistant saves its booking.<br>4. Repeat with a reschedule | Save fails with HTTP 409 and "Esse horário acabou de ser ocupado. Escolha outro." (This time slot was just taken. Choose another.). Only one session exists. A rescheduled session stays at its old time. The assistant gets the reason "reservado" (booked by someone else) | Error guessing | Negative | Automated – Vitest |
| AGT-TC-02 | Public booking without consent | Clinic with a published service on the public booking page | 1. Fill all fields with valid data.<br>2. Leave consent unchecked.<br>3. Submit | HTTP 400. No session and no patient record are saved | Equivalence partitioning | Negative | Automated – Supertest |
| AGT-TC-03 | Public booking inside the minimum notice | Same as TC-02 | 1. Build a booking request for 1 hour from now (the page offers no times in the next 2 hours).<br>2. Submit | HTTP 409. Nothing is saved | Boundary value analysis | Negative | Automated – Supertest |
| AGT-TC-04 | Clinic hours edges for the assistant | Hours in Brasília time: Monday 08:00–18:00; Tuesday 08:00–12:00 and 14:00–18:00 | 1. Check Monday 07:00, 12:00, 17:59, 18:00.<br>2. Check Tuesday 10:00, 13:00, 15:00 | Monday: closed, open, open, closed (18:00 is closing time, not open). Tuesday: open, closed (lunch break), open | Boundary value analysis | Functional | Automated – Vitest |
| AGT-TC-05 | Clinic hours: time zone and bad settings | Same hours as TC-04 | 1. Check Monday 22:00 in Brasília (Tuesday 01:00 UTC).<br>2. Check with no hours set.<br>3. Check with start time "oito" (eight, as a word).<br>4. Check with an inverted range 18:00–08:00.<br>5. Check a Saturday with an empty list and a Sunday not listed | All cases: closed. The day comes from the clinic's time zone, not the server's. Missing or invalid settings never open the clinic | Equivalence partitioning, error guessing | Negative | Automated – Vitest |
| AGT-TC-06 | Who can charge and who can reschedule | Users with roles admin, professional, secretary. One session | 1. As secretary, send a charge for the session.<br>2. As professional, send the same charge.<br>3. As secretary, change the session time.<br>4. As secretary, block a time in the schedule | 1–2: HTTP 403 (charging is admin only). 3: HTTP 200 and the new time is saved. 4: HTTP 200 or 201 | Decision table (role x action) | Security-permission | Automated – Supertest |
| AGT-TC-07 | A professional cannot change a colleague's note | Two professionals. A clinical note written by professional A | 1. As professional B, edit the note.<br>2. As professional B, delete the note.<br>3. As admin, edit the note | 1–2: HTTP 403. 3: allowed. Creating and changing notes writes an audit record | Equivalence partitioning | Security-permission | Automated – Supertest |
| AGT-TC-08 | Clinical note lifecycle: delete, trash, restore | Admin user. An active note | 1. Delete the note.<br>2. List notes and list the trash.<br>3. Try to edit the deleted note.<br>4. Restore it.<br>5. Try to restore an active note | 1: soft delete with audit. 2: note is in the trash only. 3: HTTP 404. 4: note is back in the lists, with audit. 5: HTTP 404 | State transition | Functional | Automated – Supertest |
| AGT-TC-09 | AI note blocked without recording consent | Patient with consent status "not requested" | 1. Send session audio for an AI note | HTTP 409, the error mentions "consentimento" (consent). No AI usage event is recorded (no AI call) | Equivalence partitioning | Security-permission | Automated – Supertest |
| AGT-TC-10 | "Pay before confirm" rule blocks manual confirmation | Clinic requires payment before confirmation. One unpaid session, one session without the rule | 1. Confirm the unpaid session by hand.<br>2. Confirm the session without the rule | 1: HTTP 409. 2: confirmed. An unpaid session stays "waiting" until the session ends | State transition | Functional | Automated – Supertest |
| AGT-TC-11 | AI session billed once when the request is retried | AI clinical note module on | 1. Send the transcript for processing with a session key.<br>2. Send it again with the same key (retry) | 2 usage events are saved, but the billable session count is 1. A session key over 200 characters gets HTTP 400 before any AI call | Error guessing | Regression | Automated – Supertest |
| AGT-TC-12 | Internal note in the inbox is never sent to WhatsApp | A WhatsApp conversation in the inbox | 1. Save an internal note.<br>2. Send a normal message (for comparison).<br>3. Save a note with only spaces.<br>4. Save a note with 2,001 characters | 1: no call to WhatsApp; the note has its own direction; conversation preview and order do not change. 2: WhatsApp is called. 3–4: HTTP 400. A secretary can write notes | Boundary value analysis, equivalence partitioning | Functional | Automated – Supertest |
| AGT-TC-13 | "Unanswered conversation" alert | Conversations with different last messages | 1. Patient wrote last, wait time above the limit.<br>2. Clinic wrote last.<br>3. Patient wrote last, wait time below the limit.<br>4. Patient wrote last (above the limit), then staff added only an internal note | Alert only in case 1 and case 4 (an internal note is not a reply). No alert in cases 2 and 3 | Decision table | Functional | Automated – Vitest |
| AGT-TC-14 | Publish an automation flow with errors | Admin and secretary users. A draft flow with an invalid graph | 1. As admin, publish the invalid flow.<br>2. As admin, run "validate" only.<br>3. As secretary, list and create flows | 1: HTTP 400 with the errors in Portuguese; nothing is published. 2: errors returned, nothing published. 3: HTTP 403 | Equivalence partitioning | Negative | Automated – Supertest |
| AGT-TC-15 | Secretary menu hides clinical items | Target environment URL and a secretary test account are set | 1. Open the app.<br>2. Sign in as secretary.<br>3. Look at the side menu | "Sair" (Sign out), "Pacientes" (Patients), "Agenda" (Schedule) and "Financeiro" (Billing) are visible. "Dashboard" and "Nova Sessao" (New session) are not shown | Equivalence partitioning | Security-permission | Automated – Playwright |
| AGT-TC-16 | Forbidden promise is vetoed, not rewritten | Assistant replying in an open conversation, already identified | 1. Model draft: "Garanto que você vai melhorar em 3 sessões." (I guarantee you will get better in 3 sessions.)<br>2. Model draft with a diagnosis while official prices are set.<br>3. Normal draft: "Temos horário na quinta às 15h. Quer que eu reserve?" (We have Thursday at 3 pm. Do you want me to book it?) | 1–2: vetoed by the "promessa" (promise) gate. The verdict has only allowed / gate / reason, with no rewritten text; the reason goes back to the model. 3: allowed | Decision table | AI behavior | Automated – Vitest |
| AGT-TC-17 | First message must say it is an assistant | New conversation, the assistant has not identified itself | 1. First draft without self-identification.<br>2. First draft: "Oi! Sou um robô da clínica, posso ajudar?" (Hi! I am the clinic's robot, can I help?)<br>3. Later draft without self-identification.<br>4. Reply to a person who wrote first, without self-identification | 1: vetoed by the "divulgacao" (disclosure) gate. 2: allowed (regression: the accent at the end of "robô" once broke this check). 3: allowed (no repeat on every turn). 4: vetoed | State transition | AI behavior | Automated – Vitest |
| AGT-TC-18 | Staff took over: assistant stays silent | A staff member took over the conversation | 1. Takeover ends in 1 hour; patient writes.<br>2. Takeover ended 1 hour ago; patient writes | 1: action "nao_fazer_nada" (do nothing) and no AI model call. 2: the assistant answers | State transition | AI behavior | Automated – Vitest |
| AGT-TC-19 | Risk message: silence and urgent handoff | Assistant on. A test list of 21 synthetic risk phrases and 4 normal phrases | 1. Send each risk phrase.<br>2. Send each normal phrase (for example "tem horário na quinta?" = do you have a time on Thursday?) | 1: action "silenciar_e_chamar_humano" (go silent and call a human), reason "risco" (risk), urgency "imediata" (immediate), no AI model call, conversation paused 72 hours, staff text says the assistant did NOT reply. 2: no handoff | Equivalence partitioning | AI behavior | Automated – Vitest |
| AGT-TC-20 | Risk alert delivery when a channel fails | Clinic with a risk alert phone and admin emails | 1. Direct WhatsApp send fails.<br>2. Both WhatsApp and email fail.<br>3. Second alert in the same conversation within the time window, after a delivered alert.<br>4. Same, after a failed alert | 1: alert goes to the send queue. 2: saved as not delivered. 3: suppressed. 4: not suppressed. The WhatsApp alert has no patient name and no message text | Error guessing | AI behavior | Automated – Vitest |
| AGT-TC-21 | Daily AI budget boundary | Clinic with a daily AI spend cap | 1. Spend just below the cap; request a turn.<br>2. Spend equal to the cap; request a turn.<br>3. Cap missing, zero or negative | 1: allowed, provider called once. 2: blocked with reason "teto_diario" (daily cap), no HTTP call to the provider, backup provider not tried. 3: the default cap applies; zero never means "no limit" | Boundary value analysis | AI behavior | Automated – Vitest |
| AGT-TC-22 | Per-user request limit boundary | Two signed-in users on the same IP. Limit L per 15-minute window | 1. User A sends L requests.<br>2. User A sends request L+1.<br>3. User B sends a request | 1: all HTTP 200. 2: HTTP 429, a Retry-After header between 1 and 900 seconds, message "Muitos pedidos em pouco tempo. Aguarde alguns segundos e tente de novo." (Too many requests in a short time. Wait a few seconds and try again.). 3: HTTP 200 (the limit is per person) | Boundary value analysis | Negative | Automated – Supertest |
| AGT-TC-23 | AI provider fallback by error type | Primary provider OpenAI, backup Anthropic with a key | 1. Primary returns 429.<br>2. Primary returns 400.<br>3. Backup has no key, primary fails.<br>4. The primary attempt reaches the daily cap | 1: backup answers; two usage rows are saved. 2: no backup call. 3: primary error is returned. 4: backup not tried. Both providers get the same temperature | Decision table | AI behavior | Automated – Vitest |
| AGT-TC-24 | Clinical note output schema and fallback | Clinical note JSON schema | 1. Walk every object in the schema.<br>2. Check the request format.<br>3. Simulate "model does not support json_schema".<br>4. Simulate a quota error and a wrong key error | 1: every object forbids extra fields and requires all its fields; risk level is one of none / low / moderate / high. 2: strict mode is on. 3: recognized as a format refusal (the code then retries in plain JSON mode, so the therapist still gets a note). 4: not treated as a format refusal | Equivalence partitioning | AI behavior | Automated – Vitest |
| AGT-TC-25 | Offered slot is taken before the patient says yes | Release deployed, two test phone numbers, assistant on | 1. From number 1, ask the assistant for a time.<br>2. Book that same time from the staff schedule.<br>3. From number 1, reply "pode ser" (that works).<br>4. Repeat with a two-option offer and with a reschedule | The assistant says "outra pessoa já reservou esse horário" (someone else already booked this time) and offers other free times in the same reply. It books only after the new choice | Error guessing | AI behavior | Manual |

## Decision table: assistant veto chain

The chain checks every draft message from the assistant. Gates run in a fixed order. The **first** veto stops the chain. "Reply" means the message answers something the person just wrote; "Initiative" means the clinic starts the contact.

| Rule | Opt-out (asked to stop) | Reply or initiative | Staff takeover active | Inside service hours | Forbidden promise in text | Call-back promise without an open case | First message without self-ID | Result |
|---|---|---|---|---|---|---|---|---|
| R1 | Yes | Initiative | any | any | any | any | any | Veto: `opt_out` |
| R2 | Yes | Reply | No | any | No | No | No | Send (a person who writes is still served) |
| R3 | No | any | Yes | any | any | any | any | Veto: `humano_assumiu` (human took over) |
| R4 | No | Initiative | No | No | any | any | any | Veto: `janela_de_atendimento` (service window) |
| R5 | No | Reply | No | No | No | No | No | Send (replies outside hours are allowed) |
| R6 | No | any | No | Yes | Yes: price without official prices, diagnosis, or guarantee | any | any | Veto: `promessa` (promise) |
| R7 | No | any | No | Yes | Price text, and the clinic wrote official prices | No | No | Send (only the price check is conditional) |
| R8 | No | any | No | Yes | No | Yes | any | Veto: `retorno` (call-back) |
| R9 | No | any | No | Yes | No | No | Yes | Veto: `divulgacao` (disclosure) |
| R10 | No | any | No | Yes | No | No | No | Send |

Extra rules tested:
- Opt-out and a forbidden promise together: the result is `opt_out` (first veto wins, so the log shows the real reason).
- An invalid takeover date does not silence the assistant by mistake.
- The gate list and its version number are locked by a test. Changing the order on purpose makes that test fail first.
- One veto is normal: the model rewrites. Two vetoes in a row hand the conversation to staff.

Source: `agente-cadeia-de-veto.test.ts` (29 test cases), `agente-casos-puro.test.ts` (call-back gate), `agente-handoff.test.ts` (two vetoes).

## Boundary values summary

| Boundary | Values tested | Expected | Source test |
|---|---|---|---|
| Clinic closing time (18:00 local) | 17:59 / 18:00 | open / closed | `agente-horario.test.ts` |
| Before and after opening (08:00 local) | 07:00 / 12:00 | closed / open | `agente-horario.test.ts` |
| Daily AI spend cap | just below cap / equal to cap | allowed / blocked | `agente-orcamento.test.ts` |
| Per-user request limit L | request L / request L+1 | HTTP 200 / HTTP 429 | `limite-de-uso-por-usuario.test.ts` |
| Reminder-decline context (valid 10 minutes, one turn) | read at 9 min / read at 11 min / second read | valid / expired / empty | `agente-recusa-de-confirmacao.test.ts` |
| Takeover pause | ends in +1 h / ended -1 h | silent / answers | `agente-turno.test.ts` |
| Handoff pause length | normal / risk | 4 hours / 72 hours | `agente-handoff.test.ts` |
| Internal note length | 2,001 characters / only spaces | HTTP 400 / HTTP 400 | `inbox-internal-notes.test.ts` |
| Public booking notice | 1 hour from now | HTTP 409 | `public-booking.test.ts` |

## Traceability

| ID | Test file | Test cases in file |
|---|---|---|
| AGT-TC-01 | `agenda-corrida-do-horario.test.ts` | 22 |
| AGT-TC-02, 03 | `public-booking.test.ts` | 21 |
| AGT-TC-04, 05 | `agente-horario.test.ts` | 11 |
| AGT-TC-06 | `appointment-role-boundaries.test.ts` | 4 |
| AGT-TC-07 | `note-integrity.test.ts` | 5 |
| AGT-TC-08 | `note-integrity.test.ts`, `note-trash.test.ts` | 5 + 5 |
| AGT-TC-09 | `security-hardening.test.ts` | 168 |
| AGT-TC-10 | `trava-de-pagamento.test.ts` | 21 |
| AGT-TC-11 | `ai-billing-session-dedup.test.ts` | 6 |
| AGT-TC-12 | `inbox-internal-notes.test.ts` | 9 |
| AGT-TC-13 | `conversa-nao-respondida.test.ts` | 18 |
| AGT-TC-14 | `automacoes-rotas.test.ts` | 20 |
| AGT-TC-15 | `tests/e2e/smoke.spec.ts` | 4 |
| AGT-TC-16, 17 | `agente-cadeia-de-veto.test.ts` | 29 |
| AGT-TC-18 | `agente-turno.test.ts` | 44 |
| AGT-TC-19 | `agente-handoff.test.ts`, `agente-turno.test.ts` | 41 + 44 |
| AGT-TC-20 | `agente-alerta-de-risco.test.ts` | 34 |
| AGT-TC-21 | `agente-orcamento.test.ts`, `chamada-com-reserva.test.ts` | 32 + 18 |
| AGT-TC-22 | `limite-de-uso-por-usuario.test.ts` | 5 |
| AGT-TC-23 | `chamada-com-reserva.test.ts`, `provedor-contrato.test.ts` | 18 + 5 |
| AGT-TC-24 | `nota-clinica-schema-estrito.test.ts` | 10 |
| AGT-TC-25 | Real-world check in pre-release note v0.3.2 | Manual |

Test case counts come from `npx vitest list --json` (2026-10-16). Test files are in `tests/backend/` unless shown otherwise.

---

[← Back to portfolio](../../README.md)
