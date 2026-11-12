# legacy-parity

A [Claude Code](https://claude.com/claude-code) skill for porting legacy stored procedures and screens to a new system, and proving that the new version shows the same rows and values.

Use it when you migrate old pages (classic ASP, ASP.NET, JSP) and stored procedures to a new stack, review a ported query, or answer a QA report that says "this differs from the old system". It triggers on English and Korean requests (for example "compare with legacy", "레거시와 동일하게", "SP 이식").

## What it makes Claude do

1. **Decide the kind of task**: a port (legacy is the spec for data), a re-planned screen (legacy is only a reference), or a parity check (report, don't change code).
2. **Work from primary sources**: the page that is actually served, full SP definitions read from the database catalog, and the new code from query to UI. Every claim cites a file and line or an object name.
3. **Port by copying, not rewriting**: start from the legacy definition and list every change with its reason. Do not "fix" legacy behavior such as `LIKE` wildcards, join types, missing `ORDER BY`, or which duplicate row wins.
4. **Verify with data**: run legacy and new with the same parameters and compare totals, counts per filter, first-page order and values, and boundary cases.
5. **Report every difference** as Same, Intended, Defect, or Unverified.

Legacy objects and source are treated as read-only.

## Files

| File | Contents |
|---|---|
| `SKILL.md` | The workflow above |
| `references/sp-porting.md` | Traps that cause silent regressions when porting stored procedures (end-of-day boundaries, string-coded flags, integer division, catalog truncation, mapper pitfalls) |
| `references/screen-parity.md` | Checklist for comparing search, list, detail, download, and save behavior |
| `references/legacy-source.md` | Finding the code that actually runs: includes, iframes, legacy encodings, unreliable searches |

The SQL examples are T-SQL (SQL Server). The workflow and most checklist items apply to other databases as well.

## Install

For your user (all projects):

```bash
git clone https://github.com/jjorae/legacy-parity.git ~/.claude/skills/legacy-parity
```

For one project, copy or clone it into `<project>/.claude/skills/legacy-parity`.

Claude Code loads the skill automatically when a request matches its description. You can also ask for it by name.

## License

MIT
