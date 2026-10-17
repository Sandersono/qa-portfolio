# PR #1473: AI Agent "Publish" Button Stays Disabled

**TL;DR:** After saving a draft, the "Publish" button of an AI agent stayed gray, so the new agent version could not go live from the screen. The cause was a string comparison that treated two equal objects as different.
I replaced it with a canonical comparison and added 5 test assertions. The maintainer confirmed the cause on `main`, and the PR was merged.

Related documents: [Overview](overview.md) · [PR #2324](pr-2324-webhook-retention.md) · [PR #2327](pr-2327-followup-go-live.md)

---

## Summary

| Field | Value |
|---|---|
| PR | [melgarafael/DeskcommCRM#1473](https://github.com/melgarafael/DeskcommCRM/pull/1473) |
| Status | Merged into `main` on 2026-09-23 (opened on 2026-09-22) |
| Area | AI agent editor |
| My commit | `5b0d4ae3b`: 3 files, +143 / −1 |
| Severity (my assessment) | High. The owner cannot publish a new agent version from the screen. No data is lost, and the only workaround is outside the screen. |

## Defect report

**Title:** The "Publish" button stays disabled after saving the draft of an existing AI agent.

**Environment:** Production installation, PostgreSQL 15. The agent's follow-up settings were saved before the `send_window` field existed.

**Steps to reproduce:**
1. Open an existing AI agent whose follow-up settings were saved before the `send_window` field existed.
2. Edit the agent instructions.
3. Click "Save draft".
4. Try to click "Publish".

**Expected result:** "Publish" is enabled, because the saved draft matches the screen.

**Actual result:** "Publish" stays gray with the hint "Salve o rascunho antes de publicar" ("Save the draft before publishing"). Saving again repeats the cycle. The agent stays on the old version.

## Root cause

The form decided if there were unsaved changes like this:

```ts
const dirty = JSON.stringify(form) !== JSON.stringify(baseline);
```

This asks: "Are the two strings identical, byte by byte?" Two differences that are **not edits** made the answer always "no":

1. **The server fills in fields.** The loaded version had `followup = {enabled, flow_pointer_ids}`. On save, the schema stores `{enabled, flow_pointer_ids, send_window: null}`. The form never saw the new key, but the baseline came back with it.
2. **PostgreSQL `jsonb` reorders keys.** The screen sends the keys in one order, and the database returns them in another. The content is the same, but the string is different.

The form was not re-synced after the page refresh, so the difference never went away.

## Fix

The question became the right one: **"Would saving change anything?"** The code now compares the payloads that would be saved, in a canonical form:

- Keys are sorted at every level, so the `jsonb` key order no longer matters.
- `null` and a missing key count as the same thing. This is exactly what the server fills in.
- Array order is kept. In a list of capabilities, order is content, not noise.

The comparison moved out of the UI component into a pure function, `lib/ai/agents/mesmo-rascunho.ts` ("same draft"), so it can be unit tested.

## Regression tests

`lib/ai/agents/mesmo-rascunho.test.ts` has 5 assertions:

- **The two production cases** must count as "same draft": different `jsonb` key order, and a field that the server filled in with `null`.
- **Contrast cases** must still count as an edit: changing the prompt text, and changing the order of the trigger words. These prove the fix did not turn into "everything is equal".

Technique: equivalence partitioning. One class is "not an edit" (key order, `null` vs. missing). The other class is "real edit" (text change, array order change). I tested both classes.

## Independent verification (from the maintainer's review)

- Confirmed the cause on `main`: the form compares with `JSON.stringify`, the form is not re-synced after the refresh, and validation stores `send_window` with a `null` default.
- Checked that treating `null` as missing is safe for every nullable field in this schema.
- Ran my function with extra assertions of their own. These still count as edits: switching to a credential and back, turning on the send window, and comparing `false`, `0` or `""` with a missing field.
- Found no conflict with `main` or with another open PR that changes the same file.
- After the required checks passed, the PR merged automatically.

## What the review added

The project needs a small release-note file in `.changes/` for every user-visible change. This rule was not in the contributing guide. The maintainer added the file in a separate commit and credited me in the note.

## What this shows

- Reproducing a UI defect that only happens with real, older data
- Finding a root cause in data serialization, not in the UI itself
- Contrast tests that guard against over-fixing

[← Back to portfolio](../../README.md)
