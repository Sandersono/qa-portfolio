# GBP Manager: Representative Test Cases

**TL;DR:** 25 test cases taken from the real Playwright journeys, server tests and homologation scripts of GBP Manager.
They cover functional, negative, permission, mobile (Pixel 7 profile) and AI-output scenarios. 22 are automated and 3 are manual sign-offs.
The product UI is in Portuguese, so I quote the Portuguese text and add the English meaning in parentheses.

Related documents: [Overview](overview.md) · [QA strategy](qa-strategy.md) · [Bug case study](bug-case-study.md) · [Release validation](release-validation.md)

---

## How to read this table

- **Execution:** "Automated – Playwright" runs on two projects, Desktop Chrome and Pixel 7, unless the case says otherwise. "Automated – node:test" runs in the server or prospecting test suite. "Manual sign-off" is a homologation step with evidence.
- AI cases use a fake AI client in the test. They never call the real OpenAI API.
- The source file for each case is in the [traceability table](#traceability) at the end.

## Test cases

| ID | Title | Preconditions | Steps | Expected result | Technique | Type | Execution |
|---|---|---|---|---|---|---|---|
| GBP-TC-01 | Client pages load without errors | Logged in as a test client user | 1. Open `/dashboard`.<br>2. Repeat for `/hours`, `/local-ranking`, `/ajuda`.<br>3. Collect page errors and console errors. | Page body is visible. No "Application error", no "Cannot read properties", no "Erro inesperado na API" (Unexpected API error). No JavaScript page errors. No console errors except favicon or resource loads. | Equivalence partitioning | Regression | Automated – Playwright |
| GBP-TC-02 | Mobile menu opens and goes to Local Map | Client user. Pixel 7 profile. Viewport 390 × 844. | 1. Open `/dashboard`.<br>2. Check the heading "Painel Completo" (Full Dashboard).<br>3. Tap "Abrir menu" (Open menu).<br>4. Check the link "Perfil e Horários" (Profile & Hours) is visible.<br>5. Tap "Mapa Local" (Local Map). | URL is `/local-ranking` and the page shows "Mapa Local". On Desktop Chrome the test is skipped by design. | State transition | Mobile | Automated – Playwright (Pixel 7 only) |
| GBP-TC-03 | Manual support ticket when external delivery fails | Client user. Ticket API mocked: ticket saved, external delivery status "failed". | 1. Open `/ajuda`.<br>2. In "Fale com o suporte" (Contact support), choose type "technical" and priority "high".<br>3. Fill "Assunto" (Subject) and "Descrição" (Description).<br>4. Click "Abrir chamado" (Open ticket). | Ticket number #909 is shown. The page warns about the "entrega externa" (external delivery) failure. The ticket request was sent and the ticket is shown, so the failure does not hide it. | Error guessing | Negative | Automated – Playwright |
| GBP-TC-04 | Billing-restricted client is redirected from blocked pages | Client user. Session marked as billing-restricted (mocked). Desktop Chrome and Pixel 7. | 1. Open `/hours`.<br>2. Open `/local-ranking`.<br>3. Open `/posts`. | Each page redirects to `/configuracoes-pagamentos` and shows "Regularização financeira" (Financial regularization). | Decision table | Security-permission, Mobile | Automated – Playwright |
| GBP-TC-05 | Billing-restricted client keeps access to help and tickets | Same as GBP-TC-04 | 1. Open `/dashboard`.<br>2. Open `/ajuda`.<br>3. Open `/chamados`. | `/dashboard` goes to `/configuracoes-pagamentos`. `/ajuda` stays on `/ajuda`. `/chamados` goes to `/ajuda` and shows tickets or human support content. | Decision table | Security-permission | Automated – Playwright |
| GBP-TC-06 | Superadmin pages load without errors | Logged in as a test superadmin | 1. Open `/superadmin/clients`, `/superadmin/financeiro`, `/operations`, `/automations`, `/access`.<br>2. Collect page and console errors. | Same health checks as GBP-TC-01 on each page. | Equivalence partitioning | Regression | Automated – Playwright |
| GBP-TC-07 | Superadmin menu navigation | Superadmin. Desktop Chrome and Pixel 7. | 1. Open `/superadmin/clients`.<br>2. Open the menu if it is collapsed (mobile).<br>3. Click "Financeiro" (Finance).<br>4. Open the menu again if needed and click "Operações" (Operations). | URL changes to `/superadmin/financeiro`, then to `/operations`. | State transition | Functional, Mobile | Automated – Playwright |
| GBP-TC-08 | Signup page shows the three steps | Visitor, not logged in | 1. Open `/signup`. | Button "Conectar com Google" (Connect with Google) is visible. Text "Escolher unidades" (Choose locations) and "Pagamento" (Payment) is visible. No JavaScript errors. | Equivalence partitioning | Functional | Automated – Playwright |
| GBP-TC-09 | Google popup returns locations without page reload | Google OAuth URL and popup mocked. The popup returns 2 locations. The follow-up candidates API returns an empty list. | 1. Open `/signup`.<br>2. Click "Conectar com Google".<br>3. Wait for the popup message. | "Conta Google conectada" (Google account connected) is shown. Both locations are listed and checked. The empty follow-up reload does not erase them. The reload was called. | State transition | Functional, Regression | Automated – Playwright |
| GBP-TC-10 | Asaas checkout return popup does not open the dashboard | New browser context. Window is marked as the checkout popup. | 1. Open `/signup?checkout=success&clientId=1` in the popup window. | Shows "Pagamento recebido" (Payment received) and asks the user to go back to the "janela principal" (main window). No "Entrar no painel" (Open dashboard) and no "Acompanhar Pedido" (Track order). | Error guessing | Negative | Automated – Playwright |
| GBP-TC-11 | Service description limit: boundary at 300 characters | None (pure function) | 1. Send descriptions with 1, 150, 299 and 300 characters.<br>2. Send descriptions with 301, 320 and 500 characters. | 1 to 300 characters: returned intact (the limit is inclusive). 301 or more: rejected with HTTP 400 and code `PROFILE_ITEM_DESCRIPTION_TOO_LONG`. The message gives the real length and the limit: "A descrição do item deve ter até 300 caracteres; esta tem 301…" (The item description must have up to 300 characters; this one has 301…). Text is never returned cut. | Boundary value analysis | Negative | Automated – node:test |
| GBP-TC-12 | Description is measured after normalization; empty is not an error | None | 1. Send a text longer than 300 characters only because of double spaces.<br>2. Send `undefined`, `null`, the number 42, an empty string and a string with only spaces. | 1: accepted, with double spaces collapsed to one. 2: each returns `null`, with no error. | Equivalence partitioning | Functional | Automated – node:test |
| GBP-TC-13 | Service name keeps Portuguese accents when published to Google | None | 1. Build the Google label for real service names such as "Fabricação de estofados" (Upholstery manufacturing) and "Venda de sofás" (Sofa sales).<br>2. Build it for a name with emoji, a line break and control characters.<br>3. Build it for a 200-character name. | 1: name is unchanged, accents kept. 2: emoji and control characters removed, line breaks become spaces. 3: label is cut to 120 characters. | Equivalence partitioning | Regression | Automated – node:test |
| GBP-TC-14 | Visible text accent guard | None | 1. Scan all UI strings in the translation helper, the server output files (e-mail, alerts, API error messages, reports) and the prospecting UI.<br>2. Compare each word with a list of words whose form without accent is always wrong. | No finding. Control checks pass: the scan finds hundreds of strings, and correctly accented words are not flagged. | Error guessing | Regression | Automated – node:test |
| GBP-TC-15 | AI turned off: deterministic local rank insight | AI disabled in the fake dependencies | 1. Request a local rank insight. | `source` is "fallback". Exactly 3 actions, sorted by impact (high, then medium, then low). The AI client is never called. | Decision table | AI output | Automated – node:test |
| GBP-TC-16 | Unsafe AI phrase is blocked and the fallback is used | AI enabled. The fake AI returns this explanation: "Garantimos o 1º lugar no Google em 30 dias." (We guarantee 1st place on Google in 30 days.) | 1. Request a local rank insight.<br>2. Repeat with safe explanation text, but one action rationale says "Ranking garantido no topo em poucos dias." (Guaranteed top ranking in a few days.) | Both cases: `source` is "fallback" with 3 safe actions. The unsafe text is never returned. | Decision table | AI output | Automated – node:test |
| GBP-TC-17 | Invalid AI output goes to the fallback | AI enabled | 1. Fake AI throws a network error.<br>2. Fake AI returns malformed JSON.<br>3. Fake AI returns only 2 actions.<br>4. Fake AI returns impact "extremo" (extreme). | Every case returns the fallback with 3 actions. The function never throws. | Error guessing | AI output, Negative | Automated – node:test |
| GBP-TC-18 | Safety filter: no false positive, and unsafe text in external data | None | 1. Check the text "Desktop 1 unidade em estoque na loja física." (Desktop 1 unit in stock at the store.)<br>2. Build the fallback when the leading competitor's Google name is "Garantimos o 1º lugar" (We guarantee 1st place). | 1: accepted ("top 1" inside "Desktop 1" is not a ranking promise). 2: the explanation does not include the competitor name and passes the safety filter. | Error guessing | AI output, Security-permission | Automated – node:test |
| GBP-TC-19 | Text sent to the AI is never cut in the middle of an emoji | None | 1. Build a text of 179 letters "a", then the emoji U+1F525, then "bcdef". Cut it at 180.<br>2. Cut "ab" + U+1F525 + "cd" at 3 and at 2. Cut the emoji alone at 1 and at 0.<br>3. Cut 300 letters "a" at 180. | 1: no lone surrogate (the old cut would break here). 2: "ab" + emoji, "ab", the emoji, and an empty string. 3: same result as the old cut, so normal prompts do not change. | Boundary value analysis | AI output, Regression | Automated – node:test |
| GBP-TC-20 | Prospecting report blocks false promises and sales words | None | 1. Check the client report text "Garantimos a primeira posição no Google." (We guarantee the first position on Google.)<br>2. Check a client report text with checkout and "assine agora" (subscribe now).<br>3. Check consultative text, such as advice to talk with the owner about the profile.<br>4. Check the same sales text in the internal summary. | 1 and 2: rejected. 3: accepted. 4: the internal summary allows sales words but still blocks promises and secrets. | Decision table | AI output | Automated – node:test |
| GBP-TC-21 | Prompt fingerprint for traceability | None | 1. Fingerprint the same prompt template twice.<br>2. Change one word in the master prompt.<br>3. Add the location's extra guideline.<br>4. Add empty or space-only parts. | 1: same 12-character hex value. 2 and 3: the value changes. 4: the value does not change. | Equivalence partitioning | AI output | Automated – node:test |
| GBP-TC-22 | Sentry filter reports only real defects | None | 1. Pass errors with status 400, 401, 403, 404, 409, 413, 422 and 429.<br>2. Pass status 500, status 502, an Error without status, and a plain string.<br>3. Pass three expected application error codes, then an unknown code. | 1: not reported. 2: reported. 3: expected codes not reported; the unknown code is reported. | Decision table | Functional | Automated – node:test |
| GBP-TC-23 | Real Google OAuth on the published domain | Real Google account with real Business Profiles. Product at `app.gbpmanager.com.br`. | 1. Open signup.<br>2. Click "Conectar com Google" and authorize.<br>3. Check that the locations appear without a page reload and are selected.<br>4. Continue to the terms and payment step. | Same result as GBP-TC-09 with the real provider. The popup closes. The Google connection is kept after the callback. | Exploratory | Functional | Manual sign-off |
| GBP-TC-24 | Asaas sandbox checkout and webhook | Asaas sandbox customer. Signup done up to payment. | 1. Pay in the Asaas sandbox checkout.<br>2. Wait for the automatic return.<br>3. Wait for the payment webhook. | Access is released automatically after the payment is confirmed. No duplicate customer or charge. Any error shows a friendly message. | State transition | Functional | Manual sign-off |
| GBP-TC-25 | Mobile billing portal is usable on a phone | Mobile viewport. Client user restricted by billing. | 1. Open the billing regularization portal.<br>2. Check "Resumo rápido" (Quick summary): status, due date and amount.<br>3. Tap "Abrir fatura agora" (Open invoice now).<br>4. Tap "Atualizar liberação" (Refresh access status). | No horizontal zoom is needed. The invoice opens in a new tab when a link exists. The status reloads. Buttons are full width and do not cover error or success messages. Desktop keeps its normal buttons. | Exploratory | Mobile | Manual sign-off |

## Techniques as I applied them

| Technique | How I used it here | Cases |
|---|---|---|
| **Boundary value analysis** | I test exactly at the limit and one step over it. Service descriptions: 299, 300 (pass) and 301 (fail). The 300 limit came from a real defect: the form allowed 320 characters, but publishing cut at 300. For AI input, the boundary is the cut position: an emoji that starts at character 180 of a 180-character cut. | 11, 19 |
| **Equivalence partitioning** | I split inputs into classes and test one value per class. Pages: client pages vs superadmin pages vs public pages. Description input: missing, empty, spaces only, valid, too long. Service names: accented, emoji, control characters, too long. | 01, 06, 08, 12, 13, 21 |
| **Decision table** | I list conditions and their combinations, then test each rule. See the AI table below. Also: billing status × page type (blocked page or always-allowed page), report audience × content type, Sentry status code × error code. | 04, 05, 15, 16, 20, 22 |
| **State transition** | I test moves between states. Mobile menu: closed, open, new page. Signup: not connected, popup open, connected with locations. Checkout: payment pending, paid, access released. | 02, 07, 09, 24 |
| **Error guessing** | I use past defects and experience to guess weak points: external delivery fails, empty reload after the popup, malformed AI JSON, unsafe words inside a competitor's name, "top 1" inside another word. | 03, 10, 14, 17, 18 |
| **Exploratory** | Short, focused sessions with real providers and real devices, guided by the homologation script and its acceptance criteria. I write down what I saw and any pending item. | 23, 25 |

### Decision table: local rank insight with AI

| Rule | AI enabled? | AI call succeeds? | JSON valid with 3 actions and known impact? | Passes safety filter? | Result |
|---|---|---|---|---|---|
| 1 | No | – | – | – | Fallback |
| 2 | Yes | No | – | – | Fallback |
| 3 | Yes | Yes | No | – | Fallback |
| 4 | Yes | Yes | Yes | No | Fallback |
| 5 | Yes | Yes | Yes | Yes | AI insight shown |

Every rule has at least one automated test (GBP-TC-15, 16 and 17).

## Traceability

| Case | Source |
|---|---|
| GBP-TC-01 to 10 | `tests/e2e/gbp-user-journeys.spec.ts` (Playwright journeys) |
| GBP-TC-09, 23 | Also homologation script v0.9.29 (Google popup at signup) |
| GBP-TC-11, 12 | `server/descricao-de-item-cabe-na-publicacao.test.ts` |
| GBP-TC-13 | `server/nome-de-servico-mantem-acento.test.ts` |
| GBP-TC-14 | `server/acentuacao-do-texto-visivel.test.ts` |
| GBP-TC-15 to 17 | `server/local-rank-insight-ai.test.ts` |
| GBP-TC-18 | `server/local-rank-insight-fallback.test.ts` |
| GBP-TC-19 | `server/corte-de-texto-nao-quebra-emoji.test.ts` |
| GBP-TC-20 | `prospeccao-gbp/src/lib/reportSafety.test.ts` |
| GBP-TC-21 | `server/rastro-do-prompt.test.ts` |
| GBP-TC-22 | `server/sentry-component.test.ts` |
| GBP-TC-23, 24 | QA playbook, section "Flows that need real homologation" |
| GBP-TC-25 | Homologation script v0.9.20 (mobile billing) |

---

[← Back to portfolio](../../README.md)
