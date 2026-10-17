# PR #2324: Webhook Retention and Oversized URLs

Upstream PR: [melgarafael/DeskcommCRM#2324](https://github.com/melgarafael/DeskcommCRM/pull/2324)  
Status: **Merged**

## Defect

A retention job selected up to 500 expired webhook rows and then updated them with a PostgREST filter containing all row IDs.

With hundreds of UUIDs, the query string could exceed the gateway's URL limit and be rejected with **414 URI Too Long**.

The failure was operationally dangerous because:

- `archived_at` was never written;
- the next run selected the same batch again;
- raw webhook bodies continued accumulating;
- the failure surfaced only as a warning, so the cron itself could appear to have run.

## Root cause

The implementation used a large `.in("id", ids)` filter. The repository already documented an 8,192-byte gateway limit in another path. A batch of 500 UUIDs could create a URL above 18 KB.

The problem was therefore not the number of rows in the database operation itself. It was the **transport representation of the filter**.

## Fix and regression design

The update was changed to use the `received_at` boundary of the last selected row instead of sending hundreds of IDs in the URL.

Important edge cases included:

- preserving database microseconds instead of converting the boundary through JavaScript `Date`;
- repeating the `archived_at is null` predicate;
- allowing rows tied at the boundary timestamp while keeping the operation idempotent;
- reporting the number of rows actually returned by the update, not the number originally requested.

Regression tests asserted that the update does not send an ID list, uses the final row's timestamp without losing precision, repeats the archive predicate and counts the rows actually updated.

## Independent validation

The upstream maintainer reviewed the behavior and challenged the regression coverage by deliberately breaking individual properties of the proposed fix. The tests turned red when the implementation was changed back toward the failure modes, providing evidence that the tests were not passing vacuously.

The contribution was merged upstream.

## QA lesson

A background job can "run" and still fail its purpose. For operational workflows, QA must test the **observable outcome and failure mode**, not only whether the scheduler invoked the code.

---

[← DeskcommCRM overview](overview.md) · [← Portfolio](../../README.md)
