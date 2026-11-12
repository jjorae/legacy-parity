# Finding the legacy source

The file you find first is often not the one that runs. Confirm before you compare.

## From URL to code

- Start from the URL users open. Map it to a file in the repository of that site, not to a similarly named file in another copy.
- Several copies of a legacy site often exist (old snapshots, other teams' copies). If they differ, find out which one is deployed and say which one you used.
- Follow `#include` and `<!--#include virtual=... -->` chains. An include with the same file name can differ between repositories; resolve the path the page really uses.
- A page may only wrap an `iframe` or redirect to another application. The logic then lives in that application's repository, which may not be in your local copies. Report where it is rather than comparing the wrapper.
- Data access may be inline SQL in the page, an SP call, or a call into a compiled library. For compiled code, look for the source project of the DLL and note that the deployed binary may differ from the source.

## Reading old files

- Classic ASP and old .NET files are often in a legacy encoding (CP949/EUC-KR, Shift_JIS, Windows-1252). Convert before reading or searching, for example `iconv -f cp949 -t utf-8 file.asp`, or search with an encoding-aware tool.
- Search the converted text for the column or label shown on screen, then trace backwards to the recordset field and the SP column.

## Searching large trees

- Searching a whole mounted Windows drive from WSL or a network share can take minutes. Narrow the search to a subdirectory and to the relevant extensions, and run long searches in the background.
- An empty result is evidence only if the search finished. Check the exit code: for `grep` and `rg`, `1` means no match, while `2`, `124` (timeout), or `127` (command not found, e.g. when `rg` is a shell function wrapped in `timeout`) mean the search did not run properly.
- Summary or analysis documents are useful for finding files, but they leave out rendering details and branches. Read the original file before concluding anything about behavior.
