# Regression-Test Discipline from Commit History

This metric is a repository-history audit of my two private production SaaS projects. It is included to show how often bug-fix work is accompanied by a change to test coverage.

## Result

| Product | Commits whose subject starts with `fix` | Those commits that also changed a test file | Rate |
|---|---:|---:|---:|
| GBP Manager | 171 | 86 | 50.3% |
| Agiliza Terapias | 520 | 451 | 86.7% |
| **Combined** | **691** | **537** | **77.7%** |

Measured in October 2026.

## Method

1. Read the Git history of each product.
2. Select commits whose subject begins with `fix`.
3. For each selected commit, inspect the changed paths.
4. Count the commit as **test-touched** when at least one changed path is a test file or belongs to the product's automated test suite.
5. Divide test-touched fix commits by all fix-prefixed commits.

The combined calculation is:

```text
537 / 691 = 77.7%
```

## What the metric means

It is evidence of a development habit: when a defect is corrected, I frequently add or adjust automated coverage in the same change.

## What the metric does not mean

- A `fix` prefix does not prove that every commit represents a production defect.
- Changing a test file does not, by itself, prove that the regression test is good.
- The two repositories have different histories and conventions, so their percentages should not be compared as a productivity score.
- The commercial source repositories are private; this public portfolio documents the method and aggregate result without exposing proprietary code.

For concrete examples where regression coverage can be independently inspected, see the [DeskcommCRM contributions](deskcommcrm/overview.md) and the [third-party defect reports](open-source-bug-reports.md).

---

[← Back to portfolio](../README.md)
