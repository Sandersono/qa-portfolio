# Third-Party Open-Source Defect Reports

**TL;DR:** I have reported reproducible defects in open-source projects I use in real systems. The reports include environment, reproduction steps, expected and actual behavior, technical evidence and a proposed direction for the fix. Three of the four cases below led to merged fixes by maintainers or other contributors. The fourth was independently reproduced and consolidated by the maintainer into a broader root-cause plan.

The Krayin and claude-mem reports were written in English. The DeskcommCRM report was written in Portuguese.

> Some report drafting and code-reading work was AI-assisted, as disclosed in the original GitHub issues. The production observations, reproduction, evidence selection and validation were human-led.

## Evidence

| Report | Defect and evidence | Outcome |
|---|---|---|
| [Krayin CRM #2680](https://github.com/krayin/laravel-crm/issues/2680) | Optional blank phone data could reach `PersonRepository` as an empty filtered array and trigger HTTP 500. The report included two input variants, code-level cause, reproduction steps, expected/actual behavior and a fix direction. | Closed as completed. Maintainer PR [#2693](https://github.com/krayin/laravel-crm/pull/2693) was merged to fix #2680. |
| [Krayin CRM #2681](https://github.com/krayin/laravel-crm/issues/2681) | A product row without `product_id` passed server validation because of `sometimes|required`, then failed at the database with HTTP 500. | Contributor PR [#2682](https://github.com/krayin/laravel-crm/pull/2682) explicitly fixed #2681, added regression tests, was approved and merged. |
| [DeskcommCRM #2326](https://github.com/melgarafael/DeskcommCRM/issues/2326) | WhatsApp/AI/flow automations triggered by birthdays or appointment events could fail with `service_boundary_stale`. The report included SQL reproduction, event-by-event measurements, root-cause analysis and a proposed single-source mapping. | Contributor PR [#2330](https://github.com/melgarafael/DeskcommCRM/pull/2330) was merged. The contributor publicly credited my SQL reproduction and comparison of the two duplicated event lists as the measurements that guided the fix. |
| [claude-mem #3906](https://github.com/thedotmack/claude-mem/issues/3906) | A five-day total observer outage remained diagnostically green while an OAuth failure was persisted as a downstream SQLite constraint error. The report correlated health state, database timestamps and logs, then proposed liveness and error-priority improvements. | Another user independently reproduced the failure on Windows. The maintainer closed the issue as **not planned** because the root cause and fix sequence were consolidated into plan [#3609](https://github.com/thedotmack/claude-mem/issues/3609). |

## What this demonstrates

### Reproducibility

The reports are structured so another engineer can repeat the failure. They separate what I observed directly from what I inferred by reading code.

Examples:

- #2680 explicitly distinguishes the blank-phone case I reproduced from a second variant inferred from key preservation after filtering.
- #2681 explains how to reproduce the missing-key condition by replaying a request without relying on client-side validation.
- #2326 includes SQL reproduction against a fresh PostgreSQL baseline and a table comparing event types.
- #3906 correlates a multi-day outage with timestamps, logs and persisted health state instead of relying on a single error message.

### Expected vs. actual behavior

Each report states the intended behavior and the observed failure, including HTTP status, exception or operational consequence.

### Root-cause investigation

The reports go beyond screenshots. They trace failures through validation rules, repository code, database constraints, event-origin resolution, OAuth state and health diagnostics.

### Tracking to resolution

I did not treat filing the issue as the end of the QA task. I followed the resulting fixes or maintainer disposition:

- #2680 -> merged maintainer fix.
- #2681 -> merged contributor fix with regression tests.
- #2326 -> merged contributor fix and public attribution of the reproduction work.
- #3906 -> independent reproduction and consolidation into the maintainer's root-cause plan.

## Why this matters for software QA

These reports provide third-party evidence of the workflow expected from a software tester:

**observe -> reproduce -> isolate -> document -> communicate -> track -> verify disposition**

They also show an important distinction: a good defect report does not require the reporter to own the product or author the final fix. The tester's job is to make the defect sufficiently clear and reproducible that the team can act on it.

---

[← Back to portfolio](../README.md)
