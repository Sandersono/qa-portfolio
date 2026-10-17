# PR #1473: AI Agent Publishing State

**TL;DR:** After saving a draft, the "Publish" button of an AI agent stayed gray, so the new agent version could not go live from the screen. The cause was a string comparison that treated two equal objects as different. I replaced it with a canonical comparison and added 5 test assertions. The maintainer confirmed the cause on `main`, and the PR was merged.

| Field | Value |
|---|---|
| Upstream PR | [melgarafael/DeskcommCRM#1473](https://github.com/melgarafael/DeskcommCRM/pull/1473) |
| Status | **Merged** into `main` on 2026-09-23 (opened on 2026-09-22) |
| Area | AI agent editor |
| My commit | `5b0d4ae3b`: 3 files, +143 / −1 |
| Severity (my assessment) | High. The owner cannot publish a new agent version from the screen. No data is lost, and the only workaround is outside the screen. |

## Defect

After editing an AI agent, saving the draft and trying to publish it, the **Publish** button could remain disabled. Saving again repeated the cycle, leaving the new version unavailable through the normal screen.

### Defect report

**Environment:** Production installation, PostgreSQL 15. The agent's follow-up settings were saved before the `send_window` field existed.

**Steps to reproduce:**
1. Open an existing AI agent whose follow-up settings were saved before the `send_window` field existed.
2. Edit the agent instructions.
3. Click "Save draft".
4. Try to click "Publish".

**Expected result:** "Publish" is enabled, because the saved draft matches the screen.

**Actual result:** "Publish" stays gray with the hint "Salve o rascunho antes de publicar" ("Save the draft before publishing"). Saving again repeats the cycle. The agent stays on the old version.

## Investigation

The UI determined whether the form was dirty with a byte-level comparison:

```ts
JSON.stringify(form)!== JSON.stringify(baseline)
```

That comparison treated two semantically equivalent states as different.

Two production conditions exposed the problem:

1. The server could add a nullable field such as `send_window: null` even when the original form did not contain the key.
2. PostgreSQL `jsonb` could return object keys in a different order.

Neither condition represented a user edit, but both changed the serialized string. The form was not re-synced after the page refresh, so the difference never went away.

## Fix and regression design

The comparison was changed from "are these serialized objects identical?" to "would saving this form change what is persisted?"

The canonical comparison:

- sorts object keys recursively;
- treats an absent nullable field and `null` as equivalent where the schema supports that behavior;
- preserves array order, because array order can be meaningful;
- still detects real edits such as a changed prompt or reordered trigger words.

The comparison moved out of the UI component into a pure function, `lib/ai/agents/mesmo-rascunho.ts` ("same draft"), so it can be unit tested.

### Regression tests

`lib/ai/agents/mesmo-rascunho.test.ts` has 5 assertions:

- **The two production cases** must count as "same draft": different `jsonb` key order, and a field that the server filled in with `null`.
- **Contrast cases** must still count as an edit: changing the prompt text, and changing the order of the trigger words. These prove the fix did not turn into "everything is equal".

Technique: equivalence partitioning. One class is "not an edit" (key order, `null` vs. missing). The other class is "real edit" (text change, array order change). Both classes are tested.

## Independent validation

From the maintainer's public review:

- Confirmed the cause in the code on `main`: the form compares with `JSON.stringify`, the form is not re-synced after the refresh, and validation stores `send_window` with a `null` default.
- Checked that treating `null` as missing is safe for every nullable field in this schema.
- Ran my comparison with extra assertions of their own. The old comparison reported a difference for the production-equivalent states, and the new one treats them as equal. Real edits still count as edits: switching to a credential and back, turning on the send window, and comparing `false`, `0` or `""` with a missing field.
- Found no conflict with `main` or with another open PR that changes the same file.
- The maintainer wrote that they did not reproduce the defect through the UI; the validation was done in code and with assertions. After the required CI checks passed, the PR merged automatically.

## What the review added

The project needs a small release-note file in `.changes/` for every user-visible change. This rule was not in the contributing guide. The maintainer added the file in a separate commit and credited me in the note.

## QA lesson

A test of equality must reflect **business semantics**, not merely serialization. A technically valid representation change can become a user-facing defect when the test oracle compares bytes instead of meaning.

---

[← DeskcommCRM overview](overview.md) · [← Portfolio](../../README.md)
