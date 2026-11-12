---
name: legacy-parity
description: Port legacy stored procedures and screens to a new system and prove the new version behaves the same. Use whenever the user is migrating or rewriting legacy code (ASP/ASPX/JSP pages, stored procedures, old admin tools) and asks to port, re-implement, compare, verify, or review parity, or when a ported query is being reviewed or "improved". Also use for Korean requests such as "레거시와 비교", "레거시와 동일하게", "레거시 동치", "SP 이식", "레거시 SP 옮겨", "레거시 화면 대조", "QA에서 레거시와 다르대", "레거시에선 보였는데 값이 안 보여", even when the word "parity" never appears.
---

# Legacy parity

The goal of a migration is that the new system shows the same rows and values as the legacy system, unless a spec or the user says otherwise. Every difference you find must end up either explained (with its source) or reported. A port that "looks right" but was never compared is the most common way regressions reach QA.

## 1. Decide what kind of task this is

- **Port**: re-implement a legacy SP or screen. The legacy behavior is the spec for data: which rows, which values, which order.
- **Re-planned screen**: the user or a spec defines what the screen shows. Legacy is a reference for data meaning only. Do not add items just because legacy had them; ask first.
- **Parity check, QA difference, or missing value**: compare and report. Do not change UI or code unless asked.

If the request does not make the kind clear, ask before writing code.

## 2. Gather evidence first

Work only from primary sources. Names, analysis documents, and memory are leads, not evidence.

1. **Legacy page actually served.** Go from the URL to the file. Follow includes, iframes, and redirects; the real logic is often in another repository or an older copy. See `references/legacy-source.md`.
2. **SP definitions from the database catalog**, full text, following nested `EXEC` calls. Some catalog views truncate definitions; see `references/sp-porting.md`.
3. **New code end to end**: query or SP, mapper, DTO, API, UI rendering.

Cite every claim with `file:line` or the object name and database. If something could not be checked, say so rather than inferring it.

## 3. Porting a stored procedure

Start from the legacy definition copied verbatim, then change only what the new system needs (pagination, database qualification, parameters the new API passes). Keep a running list of each change and why; that list is the diff explanation for review.

Do not "fix" legacy behavior while porting, even when it looks like a bug. Wildcard positions in `LIKE`, join types, the absence of `ORDER BY`, which duplicate row wins, parentheses in arithmetic, and `NULL` handling all decide which rows and values users see. A well-meant improvement makes the data differ from what users and the running legacy system show, and it hides inside a "port" where nobody looks for it. If you think legacy is wrong, report it separately and let the user decide.

Never modify legacy objects or legacy source. They are shared with systems still running. Put fixes in new objects or in the application layer, or report that legacy itself would have to change.

The recurring traps (date boundaries, string-coded flags, integer division, tie-breaking, catalog truncation, mapping pitfalls) are listed in `references/sp-porting.md`. Read it before writing or reviewing a ported query.

## 4. Verifying with data

Run legacy and new with identical parameters, read-only, and compare:

1. Total row count.
2. Row count for each search option and filter value the screen offers.
3. The first page: order and values, row by row.
4. Boundary cases: an end date equal to the data date, empty filters, `LIKE` special characters, rows with `NULL` in joined columns.

Counts alone are not enough: the same count with a different order or different rows is the typical failure (for example, an `ORDER BY` added during porting). For large results, dump both sides to files and diff them with a script instead of reading them by eye.

If the test database lacks data for a case, report that, name the data needed, and stop for that case. Do not rewrite the SP against other tables to make it return something: a test of a different query says nothing about the real one.

## 5. Comparing screens

Use the checklist in `references/screen-parity.md` (search defaults, columns, formatting, sort, paging, downloads, save behavior). For a "value is missing" report, start from the legacy page's output expression, then its source column, then the new SP column, mapper, DTO, and render. The break is usually in one of the last two.

## 6. Changes that alter visible rows need consent

Changing join types, `WHERE` conditions, `NULL` handling, de-duplication or tie-breaking, or sort order changes what users see. Ask before applying such a change, even when a reviewer or linter recommends it, and show the row counts before and after.

## 7. Report

Use this structure:

```markdown
## Scope and evidence
What was compared, with files, SPs, databases, and parameters.

## Differences
| Item | Legacy (evidence) | New (evidence) | Verdict |
|---|---|---|---|

Verdict is one of: Same / Intended (cite the spec or decision) / Defect / Unverified (say why).

## Data verification
Parameters used, counts per condition, first-page comparison, boundary cases.

## Open questions
Decisions the user needs to make.
```

Keep "Unverified" honest. A search that timed out or failed is not "not found", and a summary document is not the source.
