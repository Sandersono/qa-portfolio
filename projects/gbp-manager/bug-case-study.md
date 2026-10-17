# GBP Manager: Bug Case Study

**TL;DR:** Two real production defects, written in standard defect report format.
Case 1: service names lost their Portuguese accents on public Google profiles. No error was raised, so a data scan found it, and new guard tests now prevent it.
Case 2: Sentry caught an AI request that failed when review text was cut in the middle of an emoji. A boundary test now prevents it.

Related documents: [QA strategy](qa-strategy.md) · [Test cases](test-cases.md) · [Release validation](release-validation.md) · [Overview](overview.md)

---

Note on severity and priority: the original backlog did not use a severity field. I assigned severity and priority for this write-up, with the scale in [QA strategy](qa-strategy.md#10-defect-severity-and-priority).

## Case 1: Published service names lose Portuguese accents

| Field | Value |
|---|---|
| **ID** | GBP-DEF-001 |
| **Title** | Service names are published to the public Google profile without accents ("Fabricacao" instead of "Fabricação") |
| **Severity / Priority** | S2 Major / P1 |
| **Status** | Fixed (PR #148, 2026-09-06). Related findings: see the table below. |
| **Component** | Publishing of profile services to the Google Business Profile API |
| **Environment** | Production (VPS, Docker, Node 20). Code from 2026-04-11 until the fix. Package version line 0.9.64. |
| **Detection** | Found by observation on 2026-09-05, during the first real publish of services after other fixes. No test, log, dead letter or Sentry event caught it. I then ran read-only SQL scans over 270 service records (names and descriptions). |

### Steps to reproduce

*Reconstructed from release records and commit history.*

Precondition: a business location connected to Google.

1. In the panel, open the services list of the location.
2. Create or edit a service named "Fabricação de estofados" (Upholstery manufacturing). Give it a description that also has accents.
3. Publish the services to Google.
4. Open the public Google Business Profile of the location.

### Expected vs actual

| Expected | Actual |
|---|---|
| The public profile shows "Fabricação de estofados". | The public profile shows "Fabricacao de estofados". The description, sent in the same request, keeps its accents ("Fabricação", "Sofás"). |

### Impact

- Almost every Portuguese service name has an accent. Every one published after 2026-04-11 appeared without accents on the public profile.
- To a customer, it looks like the business cannot spell. It hurts trust in the business and in my product.
- The defect was in the code for almost five months. It became visible only when services were published again, and it appeared on the first try.

### Root cause

- A helper prepared service names for Google. It decomposed the text (Unicode NFD), dropped the accent marks, and removed every non-ASCII character.
- It was added on 2026-04-11 "for API safety", in a series of direct commits, with no recorded error behind it.
- The next commit fixed a text encoding problem (garbled characters). Most likely the helper hid that symptom first. The real encoding fix came later, and the helper stayed.
- Proof that Google accepts accents: in the same request, the description field kept its accents. Only the name went through this helper.

### Fix

- Normalize text to Unicode NFC (composed form) instead of NFD.
- Block only what can really break or pollute a label: control characters, line separators and emoji.
- Keep the 120-character limit.
- I did not delete the helper. Removing it without understanding it would repeat the original mistake. What it did right stays, with tests.

### Verification

- 5 new tests in `server/nome-de-servico-mantem-acento.test.ts`, including a control test that checks the test's copy of the rule against the real module.
- Mutation check: switching back to NFD makes 2 of 5 tests fail. Removing the emoji filter makes 1 of 5 fail.
- `npm run ci` passed locally.
- Data: on 2026-09-22 I corrected 22 items in the database. Since then, the Portuguese and accent scan queries return no rows.

### Related findings from the same scan

| Finding | Origin | Status |
|---|---|---|
| "rodizio" instead of "rodízio" (a restaurant service style) in 4 items | Data imported from Google as is | Fixed 2026-09-22 |
| One description starts with "ratamos…" (the "T" of "Tratamos", "We treat", is missing) | Data imported from Google as is | Fixed 2026-09-22 |
| 7 service names start with a lowercase letter | Data | Fixed (first letter capitalized) |
| New checks on 2026-09-22: missing space after a comma (5 items), spelling from before the 2009 spelling reform (1), brand name without its accent (5), all-caps name without accent (1) | Data | Fixed the same day |
| The same profile description on several locations of one brand; identical service descriptions inside one business (6 items for 3 services); near-identical service names | Content | Open. This is a content decision, to review with each business. |
| Description cut in the middle of a word: the form allowed 320 characters, but publishing cut at 300 (9 items, all written in my own form) | My code | Fixed in PR #193 (2026-09-24): one shared limit of 300 from form to publish, a character counter, and a clear error instead of a silent cut. The 9 stored texts must be shortened one by one with each business; their current status is not recorded in the documents I used. |

### Prevention

- **Accent guard test** (`server/acentuacao-do-texto-visivel.test.ts`, added in PR #161 and extended in PR #176). It scans visible text in three channels: UI strings in the translation helper, 16 server output files (e-mail, alerts, API error messages shown on screen, reports) and the prospecting UI. It compares words against a list whose form without accent is always wrong. The list grew from 24 to 550 words. Control tests make sure the scan finds hundreds of strings and does not flag correct words.
- **`audit:i18n`**: lists visible UI text that is not wrapped in the translation helper. Strict mode fails the run.
- **One shared limit** for service descriptions, with boundary tests at 300 and 301 (see [GBP-TC-11](test-cases.md#test-cases)).
- **Read-only SQL scan queries**, stored with the backlog and run at each review: missing accents, lowercase start, double spaces, garbled characters, missing space after a comma, old spelling, all caps, duplicates, and descriptions over 300 characters.
- **Fix rule by origin**: my error is fixed in code with a test. A source error is fixed by editing the item in the panel, after agreeing with the business. Duplicates between locations are a content decision.

### Lessons learned

- **A text defect does not fail; it stays.** It gives no error and no Sentry event. Only the customer sees it. Tests are not enough: I also need scans of real data.
- Do not add a "defensive" transformation without a recorded reason. Do not remove one blindly either.
- Test every output channel with real-language data (accents).
- Two numbers for one limit will drift apart. Use one constant and test at the boundary.
- A guard test needs control tests. Without them, it can pass while it checks nothing.

---

## Case 2: AI request fails when a review is cut in the middle of an emoji

*Reconstructed from release records and commit history.*

| Field | Value |
|---|---|
| **ID** | GBP-DEF-002 |
| **Title** | AI semantic profile suggestion fails with OpenAI error 400 `invalid_json` when a review is cut in the middle of an emoji |
| **Severity / Priority** | S2 Major / P1 |
| **Status** | Fixed (PR #150, 2026-09-06) |
| **Component** | AI input preparation: semantic profile suggestion (`POST /api/content-semantic-profile/suggest`) and two other AI inputs |
| **Environment** | Production (VPS, Docker, Node 20), OpenAI Responses API |
| **Detection** | Sentry issue opened on 2026-09-01, with one event. The back-end filter reports only 5xx and unhandled errors, so this was a real server failure. No test caught it. |

### Steps to reproduce

Precondition: AI is enabled. Among the 20 most recent reviews of a business, one review is longer than 180 characters and has an emoji that crosses character position 180.

1. Request an AI semantic profile suggestion for that business.
2. The server cuts each review to 180 characters with JavaScript `slice`.
3. The server sends the request to OpenAI.

Minimal reproduction (used in the test): a text of 179 letters "a", then the emoji U+1F525, then "bcdef". Cut it with `slice(0, 180)`.

### Expected vs actual

| Expected | Actual |
|---|---|
| The suggestion is generated. Long reviews are cut at a safe point. | OpenAI rejects the whole request with HTTP 400 `invalid_json` ("failed to parse JSON value"). The API request fails with a server error. What the user saw on screen is not recorded. |

### Impact

- The AI suggestion fails for any business that has such a review among its 20 most recent reviews. Retrying does not help, because the same reviews are sent again.
- The defect is rare. Many suggestion runs on the same day passed. That is why Sentry had only one event.
- Measured on all stored reviews: 0 broken at the source, 7 broken after the old cut, 0 after the new cut. The defect was created only by my cut.
- The same pattern existed in two more AI inputs: post text (cut at 220) and business context for image analysis (cut at 1,800).

### Root cause

- JavaScript `String.slice` counts UTF-16 code units, not characters.
- An emoji uses two code units (a surrogate pair). A cut between them leaves a lone surrogate.
- A lone surrogate is not valid UTF-8, so the JSON body is invalid for OpenAI.

### Fix

- A new helper, `truncateForOpenAi`, cuts by Unicode code point (with `Array.from`). It never splits a pair.
- All three AI inputs use it: reviews (180), posts (220) and business context (1,800).

### Verification

- 5 new tests in `server/corte-de-texto-nao-quebra-emoji.test.ts`.
- Control test: text without emoji is cut exactly as before. A change in length would change every prompt and the AI result with it.
- Mutation check: putting `slice` back in the helper makes 2 of 5 tests fail. Putting only the review cut back makes 1 of 5 fail.
- `npm run ci` passed locally.
- The Sentry issue status after the deploy is not recorded in the commit. That detail is unknown.

Test excerpt:

```ts
test('ÂNCORA: cortar no meio de um emoji não deixa surrogate solto', () => {
  // O caso real: avaliação que termina em emoji, cortada no limite.
  const texto = `${'a'.repeat(179)}\u{1F525}bcdef`;
  assert.ok(temSurrogateSolto(texto.slice(0, 180)), 'CONTROLE: o slice antigo precisa quebrar');
  assert.ok(!temSurrogateSolto(truncateForOpenAi(texto, 180)), 'o corte novo não pode quebrar');
});

test('o emoji inteiro é preservado ou descartado, nunca partido', () => {
  const fogo = '\u{1F525}';
  assert.equal(truncateForOpenAi(`ab${fogo}cd`, 3), `ab${fogo}`);
  assert.equal(truncateForOpenAi(`ab${fogo}cd`, 2), 'ab');
  assert.equal(truncateForOpenAi(fogo, 1), fogo);
  assert.equal(truncateForOpenAi(fogo, 0), '');
});
```

The first test proves the old cut breaks and the new cut does not, at the exact boundary (180). The second test checks that an emoji is kept whole or dropped whole, never split.

### Prevention

- An anchor test fails if any of the three AI inputs goes back to `slice`.
- Boundary tests at the cut position with a multi-unit character (see [GBP-TC-19](test-cases.md#test-cases)).
- Sentry stays on for 5xx errors. It caught what the tests missed.

### Lessons learned

- A length limit is a boundary. Test it with characters that use more than one code unit (emoji), not only with plain letters.
- A rare defect can pass many manual runs. Production monitoring is part of testing.
- Measure the scope with real data before and after the fix. Here it proved the source data was clean and the cut was the cause.
- When you find a pattern bug, search for every place with the same pattern. Here there were three.

---

[← Back to portfolio](../../README.md)
