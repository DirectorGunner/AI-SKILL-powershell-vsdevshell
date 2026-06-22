# PowerShell / VS Developer Shell — Gotchas

Recurring failure modes when writing PowerShell / VS Developer Shell scripts in this repo, and what to do instead. Read alongside `SKILL.md`.

- This repo targets Windows PowerShell 5.1; avoid PS7-only syntax (ternary, `??`, `?.`, `&&`/`||`, `ForEach-Object -Parallel`, `Join-String`), validate with `powershell.exe`, and don't assume `pwsh` is installed.
- Default file encoding is UTF-16 LE with BOM; pass `-Encoding utf8` when another tool must read a file you write.
- Don't redirect a native executable's stderr with `2>&1` in 5.1 — it wraps lines as ErrorRecords and flips `$?` even on exit 0; stderr is already captured for you.
- Initialize the Visual Studio Developer Shell / MSVC environment before invoking MSBuild or native build tools; confirm the dev shell is entered rather than assuming PATH.
- Grouping parentheses reset `$?` in 5.1; check `$LASTEXITCODE` for native-command success, not `$?`.
- If a cmdlet, parameter, or behavior is not in the references, say so and mark it `TODO` — never invent one.