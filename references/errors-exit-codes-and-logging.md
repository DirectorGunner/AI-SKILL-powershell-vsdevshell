# Errors, Exit Codes, and Logging

PowerShell cmdlet errors and native executable failures are different systems. Handle both, and end every task script with one deterministic status line.

## Terminating vs non-terminating errors

- `try`/`catch` catches TERMINATING errors only. Most cmdlet errors are non-terminating by default and sail past `catch`.
- Escalate per command with `-ErrorAction Stop`, or script-wide with `$ErrorActionPreference = 'Stop'` (scope: current scope and children; the `-ErrorAction` common parameter overrides the preference for that one command - official: about_Preference_Variables, about_CommonParameters).
- Inside `catch`, `$_` is an ErrorRecord: use `$_.Exception.Message`, `$_.ScriptStackTrace`. Catch specific exception types first: `catch [System.IO.IOException] { ... }`.
- `finally` always runs - put cleanup and final log lines there.
- `throw` creates a terminating error; `Write-Error` emits a non-terminating one (and sets `$?` to False for itself only).

```powershell
$ErrorActionPreference = 'Stop'
try {
    Get-Item -LiteralPath '<REPO_ROOT>\required.file' -ErrorAction Stop | Out-Null
} catch {
    Write-TaskLog -Path $logPath -Message ('FAILED_WITH_EVIDENCE missing file: {0}' -f $_.Exception.Message)
    exit 1
} finally {
    Write-TaskLog -Path $logPath -Message 'phase: input-verification finished'
}
```

Caution (practical tip): with `$ErrorActionPreference = 'Stop'`, stderr output from native commands redirected via `2>&1` can surface as NativeCommandError records even on success. Judge native commands by exit code (see `quoting-and-native-commands.md`).

## Native exit codes

```powershell
& $exe @argv
$exitCode = $LASTEXITCODE
if ($exitCode -ne 0) {
    throw ('Command failed with exit code {0}: {1}' -f $exitCode, $exe)
}
```

- Capture `$LASTEXITCODE` immediately; the next native command overwrites it.
- Known tool contracts: cargo fails with 101; python exits 1 on uncaught exception and 2 on usage error; `python -m py_compile` exits nonzero on syntax errors; npm propagates the script's exit code.
- Do not claim success from output-file existence when the exit code was nonzero.

## Reusable phase-marker logging pattern (PS 5.1)

```powershell
$script:TaskLogPath = Join-Path '<WORK_ROOT>' 'task.log'
function Write-Phase {
    param([Parameter(Mandatory=$true)][string]$Message)
    $line = ('[{0}] {1}' -f (Get-Date -Format 'yyyy-MM-dd HH:mm:ss'), $Message)
    Write-Host $line
    Add-Content -LiteralPath $script:TaskLogPath -Value $line -Encoding UTF8
}

Write-Phase 'START task-name'
Write-Phase 'PHASE validate-inputs'
# ... work ...
Write-Phase ('EXIT tool.exe code={0}' -f $LASTEXITCODE)
Write-Phase 'STATUS VALIDATED'
```

Log: START line, phase markers, input/output paths, external command lines in inspectable form, exit codes, and a final deterministic status. Use deterministic status terms such as `VALIDATED`, `FAILED_WITH_EVIDENCE`, `SKIPPED_BY_SCOPE`, `BLOCKED_WITH_EVIDENCE`, and `NOT_ATTEMPTED` - never vague confidence wording.
