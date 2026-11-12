# Porting stored procedures: traps

Each item below has caused a real regression in a migration. Check every ported or reviewed query against this list.

## Data semantics

- **End-of-day boundary.** When a `DATETIME` column is compared with `BETWEEN @FROM AND @TO` and the screen sends dates only, legacy pages often appended `' 23:59:59'` to the end date. Without it, every row from the last day disappears. Check how the legacy page built the parameter, not only the SP.
- **String-coded flags.** Legacy SPs often return labels instead of codes, e.g. `CASE WHEN DEL_YN = 'Y' THEN 'Withdrawn' ELSE '' END AS DEL_YN`. New code comparing that column with `'Y'` is always false. Read the `SELECT` list for the real return values; never assume from the column name.
- **Integer division and constant folding.** In T-SQL, `@money * -1 / 1100` folds `-1 / 1100` to integer `0` first, so the result is always `0`; `@money * (-1) / 1100` works. Copy parentheses and spacing exactly, especially around `money` and integer constants.
- **`LIKE` patterns.** `'%' + @v` (suffix), `@v + '%'` (prefix), and `'%' + @v + '%'` (substring) return different rows. Keep the wildcard positions and the `ESCAPE` character character for character.
- **Join types.** `INNER` to `LEFT` (or the reverse) changes the row set: orphans appear or vanish. Keep the legacy join, including joins inside subqueries.
- **Missing `ORDER BY`.** If legacy had none, do not add one silently; the screen order users know comes from the plan the legacy query produced. If paging requires an order, pick one that reproduces the legacy order on real data and report it.
- **Which duplicate wins.** Legacy pages often looped over all rows and overwrote the same variable, so the last row returned won. Reproduce that choice (for example `TOP 1 ... ORDER BY <identity> DESC`) instead of choosing a "more meaningful" row such as active-first. Document the rule you reproduced.
- **Status code meanings drift.** The same code can mean different things in an old include file and in later code. Find the version the running system uses before mapping codes to labels.

## Database catalog and tooling

- **Truncated definitions.** `INFORMATION_SCHEMA.ROUTINES.ROUTINE_DEFINITION` stops at 4,000 characters. Use `OBJECT_DEFINITION(OBJECT_ID(...))` or `sys.sql_modules.definition`.
- **Other databases.** `OBJECT_DEFINITION(OBJECT_ID('OtherDb..Proc'))` can return another object's definition or `NULL`. Join the target database's catalog instead:

  ```sql
  SELECT o.name, m.definition
  FROM OtherDb.sys.sql_modules m
  JOIN OtherDb.sys.objects o ON o.object_id = m.object_id
  WHERE o.name = 'ProcName';
  ```

- **`CREATE OR ALTER` and three-part names.** `CREATE OR ALTER PROCEDURE Db.dbo.X` fails with a misleading `Must declare the scalar variable "@FirstParam"`. Use `dbo.X` with `USE Db` before it.
- **`GO` is a client batch separator, not T-SQL.** Some tools send the whole script as one batch and fail on `GO`. Split on `GO` lines, or run each batch separately.
- **DDL with no result set.** Some query tools report an error when a statement returns no rows even though it ran. Confirm by reading the catalog afterwards, not by the tool's message.
- **Find the tables an SP really touches** by parsing its definition (and nested `EXEC`s), then confirm each table exists. Do not infer table names from SP names; the database an SP lives in is often not the database of its tables.

## Application mapping

- **Constructor mapping by column order.** Some mappers (e.g. MyBatis with Kotlin data classes that have no no-arg constructor) fill constructor arguments in result-column order, then apply named mappings. `NULL` columns are not overwritten, so another column's value stays in that field. Give every field a default value so mapping goes by name, or test with a `NULL` row.
- **Pagination conventions.** Follow the project's existing pattern. If there is none, `COUNT(*) OVER() AS TOTAL_COUNT` keeps list and count in one SP; compare the count against the legacy count query.

## Port checklist

- [ ] Definition copied from the catalog in full, nested `EXEC`s followed
- [ ] Every change from the legacy text listed with its reason
- [ ] `LIKE`, joins, `ORDER BY`, duplicate choice, parentheses unchanged
- [ ] Date parameters built the way the legacy page built them
- [ ] Return values (labels vs codes) matched by the new mapping
- [ ] Legacy objects untouched
- [ ] Data compared: total, per condition, first page order and values, boundaries
