# PowerShell VS Developer Shell References

This reference set supports portable Windows VS Code repository work using Windows PowerShell 5.1 and Visual Studio Developer Shell tooling. It collects curated, field-tested guidance and compact retrieval chunks for agents.

## Start Here

1. Read `SKILL.md`.
2. Scan `ps51-pitfalls-checklist.md` before editing or generating any PowerShell.
3. Read `powershell-5-1-compatibility.md` when writing 5.1-target syntax.
4. Read `quoting-and-native-commands.md` before invoking native executables.
5. Read `visual-studio-devshell.md` and `msbuild-msvc-build-tools.md` for Visual Studio, MSVC, or MSBuild work.
6. Read `errors-exit-codes-and-logging.md` and `files-paths-and-encoding.md` when writing logs, reports, or files.
7. Read `embedded-python-in-powershell.md` before generating Python from PowerShell.
8. Read `validation-and-ast-checks.md` before claiming any script change is done.
9. Read `vscode-repo-agent-patterns.md` before deciding where artifacts go or when updating this skill.

## Topic Files

- `powershell-5-1-compatibility.md`: PS7-only syntax table with 5.1-safe alternatives, `#Requires`, behavior differences, and the AST parse pattern for Windows PowerShell 5.1 Desktop.
- `quoting-and-native-commands.md`: Safe native invocation - call operator, argument arrays, `--%` stop-parsing token, `$LASTEXITCODE` vs `$?`, stderr semantics, portable tool examples.
- `arrays-hashtables-functions.md`: Arrays vs List[T], unary comma, `-join`/`-split` operators, ordered dictionaries, array-filtering comparison operators, function and parameter style.
- `errors-exit-codes-and-logging.md`: Terminating vs non-terminating errors, `-ErrorAction Stop`, try/catch/finally, native exit-code contracts, reusable phase-marker logging, deterministic statuses.
- `files-paths-and-encoding.md`: Path cmdlets, literal paths, long-path caution, Windows PowerShell 5.1 encoding and BOM behavior, BOM-less UTF-8 escape hatch, artifact routing.
- `visual-studio-devshell.md`: Launch-VsDevShell.ps1 vs VsDevCmd.bat, architecture arguments, vswhere discovery without PATH assumptions, multi-instance handling, environment verification checklist.
- `msbuild-msvc-build-tools.md`: MSBuild command-line patterns, PowerShell-specific quoting of switch lists, MSVC environment and architecture verification, build reporting.
- `embedded-python-in-powershell.md`: Single-quoted line arrays, safe value injection (argv/env/JSON), py_compile validation, Python invocation, encoding in both directions.
- `validation-and-ast-checks.md`: AST parse with line numbers, smoke checks, JSON/JSONL validation with cross-reference checks, exit-code assertions, output confirmation, no Git unless requested.
- `vscode-repo-agent-patterns.md`: Detecting repo conventions, placeholder roots, artifact hygiene, skill portability rules, and how to maintain this skill's metadata files.
- `ps51-pitfalls-checklist.md`: One-page preflight checklist of recurring Windows PowerShell 5.1 mistakes across syntax, native commands, Visual Studio tools, files, and reporting.

## Metadata Files

- `topics.json` maps each topic file to its warnings and applicable work.
- `corpus.jsonl` is a compact retrieval corpus for future agents. Each line is a standalone JSON chunk referencing a topic id.
