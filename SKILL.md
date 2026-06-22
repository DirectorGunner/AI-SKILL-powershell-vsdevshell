---
name: powershell-vsdevshell
description: Windows PowerShell 5.1 and Visual Studio Developer Shell guidance for any Windows VS Code repository. Use when editing, generating, reviewing, or debugging PowerShell scripts, setup scripts, build wrappers, native executable invocations (Python, Cargo, npm, MSBuild, arbitrary tools), MSBuild/MSVC workflows, embedded Python generation, file/encoding handling, logging, reports, or validation in a Windows repo.
covers:
  - artifact routing
  - powershell scripts
  - repo setup scripts
  - generated scripts
  - windows compatibility reviews
  - native executable invocation
  - quoted tool paths
  - argument arrays
  - stop-parsing token
  - stdout and stderr logging
  - powershell functions
  - script data structures
---

# Windows PowerShell 5.1 and Visual Studio Developer Shell

Use this skill whenever a Windows VS Code repository task involves PowerShell scripts, repo setup scripts, build scripts, wrapper scripts, generated scripts, native executable calls, Visual Studio Developer Shell setup, MSBuild, MSVC, embedded Python, logs, reports, prompts, or artifact routing.

This skill is portable guidance for ANY Windows repo. Adapt examples to the current repository's roots and conventions. Rules drawn from field experience are labeled as practical tips in the reference topics.

## Required workflow

1. Read the repo-local agent instructions first; they win over generic guidance.
2. Identify the repo's approved roots for work files, reports, prompts, logs, generated scripts, and temp output (`<WORK_ROOT>`, `<REPORTS_ROOT>`, `<PROMPTS_ROOT>`).
3. Write Windows PowerShell 5.1-compatible syntax unless the task explicitly targets `pwsh`; declare `#Requires -Version 5.1` where it matters.
4. Invoke native executables with the call operator and separate argument arrays; capture `$LASTEXITCODE` immediately.
5. If Visual Studio tools are needed, load the Developer Shell once at the top, then VERIFY the environment (`VSCMD_VER`, arch variables, tools, `INCLUDE`/`LIB`).
6. Validate syntax (5.1 AST parse), JSON/JSONL, generated scripts (`py_compile` for Python), exit codes, and output routing before claiming success; end with a deterministic status.

## Hard prohibitions

- No PowerShell 7-only syntax (ternary, `??`, `??=`, `?.`, `&&`/`||` chains, `ForEach-Object -Parallel`, `Join-String`) unless the task explicitly targets `pwsh`.
- Do not rely on `pwsh` being installed.
- Do not assume `vswhere.exe` is on `PATH` or that Visual Studio lives at exactly one path; do not mix Visual Studio versions or switch developer environments inside one shell.
- Do not wrap an entire native command line as one string and expect PowerShell to split it.
- Do not trust untested nested quoting; do not read `$?` after grouping parentheses in 5.1.
- Do not put task scripts, logs, reports, prompt archives, source captures, or temp files inside `.agents\skills`.
- Do not run Git commands unless the user or task explicitly asks for Git validation.
- Do not create hardlinks or symlinks unless explicitly authorized.
- Do not copy one project's internal implementation details into reusable skill docs.

## Quick patterns

```powershell
# Native call with verified exit code
$exe  = '<TOOLS_ROOT>\tool.exe'
$argv = @('--input', '<WORK_ROOT>\input file.txt')
& $exe @argv
if ($LASTEXITCODE -ne 0) { throw ('tool failed: {0}' -f $LASTEXITCODE) }

# Developer Shell: load once, verify, then build
if ([string]::IsNullOrWhiteSpace($Env:VSCMD_VER)) {
    & '<VSINSTALLDIR>\Common7\Tools\Launch-VsDevShell.ps1' -Arch amd64 -HostArch amd64 -SkipAutomaticLocation
}
if ([string]::IsNullOrWhiteSpace($Env:INCLUDE)) { throw 'MSVC environment not loaded.' }
& msbuild '<REPO_ROOT>\Project.sln' -m -p:Configuration=Release -p:Platform=x64
```

## Reference map (all topics)

Start with `references\INDEX.md`, then open the topic that matches the task:

- `references\ps51-pitfalls-checklist.md` - preflight checklist; scan before any PowerShell edit.
- `references\powershell-5-1-compatibility.md` - PS7-only syntax table with 5.1-safe alternatives; `#Requires`; AST parse pattern.
- `references\quoting-and-native-commands.md` - call operator, argument arrays, `--%`, `$LASTEXITCODE` vs `$?`, stderr semantics, tool examples.
- `references\arrays-hashtables-functions.md` - arrays/List[T], `-join`/`-split`, `[ordered]`, array-filtering comparisons, function/param style.
- `references\errors-exit-codes-and-logging.md` - terminating vs non-terminating errors, `-ErrorAction Stop`, phase-marker logging, status labels.
- `references\files-paths-and-encoding.md` - Join-Path/Test-Path/Resolve-Path, 5.1 encoding + BOM, BOM-less .NET writer, long paths, artifact routing.
- `references\visual-studio-devshell.md` - Launch-VsDevShell.ps1 vs VsDevCmd.bat, -Arch/-HostArch/-SkipAutomaticLocation, vswhere discovery, verification checklist.
- `references\msbuild-msvc-build-tools.md` - MSBuild switches (-m, -t:, -p:, -restore, -bl), MSVC environment verification, build reporting.
- `references\embedded-python-in-powershell.md` - single-quoted line arrays, argv/env/JSON value injection, py_compile, encoding both directions.
- `references\validation-and-ast-checks.md` - AST parse, smoke tests, JSON/JSONL line validation, exit-code assertions, output confirmation.
- `references\vscode-repo-agent-patterns.md` - detecting repo conventions, placeholder roots, artifact hygiene, skill metadata maintenance.

This skill is a practical checklist and navigation layer, not a replacement for the underlying tool documentation.

## Gotchas

Recurring failure modes and what to do instead live in the sibling [GOTCHA.md](GOTCHA.md).

## Inspired by

Includes original content written for this skill, informed by: Microsoft Learn (PowerShell about_* topics, the Visual Studio Developer Shell, MSBuild, and C++ command-line docs) and the Python, Rust Cargo, and npm documentation.
