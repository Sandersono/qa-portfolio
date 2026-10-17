# PR #1473: AI Agent Publishing State

Upstream PR: [melgarafael/DeskcommCRM#1473](https://github.com/melgarafael/DeskcommCRM/pull/1473)  
Status: **Merged**

## Defect

After editing an AI agent, saving the draft and trying to publish it, the **Publish** button could remain disabled. Saving again repeated the cycle, leaving the new version unavailable through the normal screen.

## Investigation

The UI determined whether the form was dirty with a byte-level comparison:

```ts
JSON.stringify(form) !== JSON.stringify(baseline)
```

That comparison treated two semantically equivalent states as different.

Two production conditions exposed the problem:

1. The server could add a nullable field such as `send_window: null` even when the original form did not contain the key.
2. PostgreSQL `jsonb` could return object keys in a different order.

Neither condition represented a user edit, but both changed the serialized string.

## Fix and regression design

The comparison was changed from "are these serialized objects identical?" to "would saving this form change what is persisted?"

The canonical comparison:

- sorts object keys recursively;
- treats an absent nullable field and `null` as equivalent where the schema supports that behavior;
- preserves array order, because array order can be meaningful;
- still detects real edits such as a changed prompt or reordered trigger words.

Regression tests covered the two production cases and contrasting cases that must remain different.

## Independent validation

The upstream maintainer reproduced the cause on the project's main branch and independently exercised the comparison with additional assertions. The maintainer confirmed that the old comparison reported a difference for the production-equivalent states while the new comparison treated them as equal, and also checked that real edits still remained edits.

The contribution was merged upstream.

## QA lesson

A test of equality must reflect **business semantics**, not merely serialization. A technically valid representation change can become a user-facing defect when the test oracle compares bytes instead of meaning.

---

[← DeskcommCRM overview](overview.md) · [← Portfolio](../../README.md)
