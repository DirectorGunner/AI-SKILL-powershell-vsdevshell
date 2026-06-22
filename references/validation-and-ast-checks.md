# Validation and AST Checks

Validate the thing the task changed before claiming success, and validate it with the engine that will run it.

## PowerShell AST parse

Use the Windows PowerShell parser for Windows PowerShell 5.1 scripts, executed under `powershell.exe`:

```powershell
$tokens = $null
$errors = $null
[System.Management.Automation.Language.Parser]::ParseFile('<WORK_ROOT>\script.ps1', [ref]$tokens, [ref]$errors) | Out-Null
if ($errors.Count -ne 0) {
    $errors | ForEach-Object { Write-Error ('{0} at line {1}' -f $_.Message, $_.Extent.StartLineNumber) }
    exit 1
}
```

List every parse error with its line number; do not stop at the first. A clean parse under `pwsh` proves nothing about 5.1 - run the validator with the target engine.

## Smoke checks

After a clean parse, run the cheapest real execution the script supports:

- `-Help` or usage mode for wrapper scripts (assert exit code 0 and expected usage text).
- `-WhatIf`, dry-run, or plan modes where implemented.
- A minimal no-op invocation otherwise.

A parse check plus a smoke check catches most generation mistakes before expensive runs.

## JSON and JSONL

```powershell
$json = Get-Content -LiteralPath '<REPORTS_ROOT>\out.json' -Raw
$null = $json | ConvertFrom-Json
```

```powershell
$lineNo = 0
Get-Content -LiteralPath '<REPORTS_ROOT>\corpus.jsonl' | ForEach-Object {
    $lineNo += 1
    if ($_.Trim().Length -gt 0) {
        try { $null = $_ | ConvertFrom-Json } catch { throw ('Invalid JSONL at line {0}' -f $lineNo) }
    }
}
```

- Report the FIRST invalid line number for JSONL - it makes the fix immediate.
- When metadata files cross-reference each other (ids, file names), validate the references too: every referenced file exists, every referenced id is defined.
- Practical tip: JSON written by 5.1 with `-Encoding UTF8` carries a BOM; non-PowerShell consumers should read it as `utf-8-sig` or the writer should use the BOM-less .NET pattern.

## Native tools and outputs

- Check `$LASTEXITCODE` for every native command; record command, arguments, and exit code.
- `python -m py_compile` for generated Python; build/test exit codes for Rust (`cargo` fails with 101) or C++ (MSBuild/`cl.exe`/`link.exe`) when in scope.
- Confirm expected output paths exist, are non-empty where appropriate, and landed in the repo-designated artifact roots.
- Use noninteractive stdin redirection only for known prompt blockers, documented.

## What not to validate

Do not run Git commands or claim Git cleanliness unless the user or task explicitly requests Git validation. When skipped on request, state exactly that it was skipped by user preference. Record every skipped validation honestly with its reason.
