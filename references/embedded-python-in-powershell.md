# Embedded Python in PowerShell

Generating Python from PowerShell breaks when quoting and interpolation are casual. Use single-quoted line arrays, inject values through argv/env/JSON files instead of string interpolation, and validate with `py_compile` before running.

## Safe line-array pattern

Single-quoted PowerShell strings are verbatim - PowerShell will not expand `$name`, backticks, or subexpressions inside them, so Python code survives intact:

```powershell
$py = @(
    'import json',
    'import sys',
    'from pathlib import Path',
    'print("START generated job")',
    'config = json.loads(Path(sys.argv[1]).read_text(encoding="utf-8-sig"))',
    'print("STATUS OK")'
)
Set-Content -LiteralPath '<WORK_ROOT>\generated.py' -Value $py -Encoding UTF8
```

Single quotes INSIDE a Python line are written by doubling them (`'it''s'`). Keep one Python statement per array element; never build Python from interpolated double-quoted PowerShell strings.

## Injecting values safely

Never interpolate values into Python source. In order of preference:

1. Command-line arguments: `& $python '<WORK_ROOT>\generated.py' $inputPath $mode` and read `sys.argv` in Python.
2. A JSON input file written by PowerShell and read by Python (`encoding='utf-8-sig'` tolerates the 5.1 BOM - practical tip).
3. Environment variables: `$Env:JOB_INPUT = $inputPath` then `os.environ['JOB_INPUT']`.

If interpolation is truly unavoidable, use `-f` format strings into single-quoted templates and validate the result file immediately.

## Validation and invocation

```powershell
& '<TOOLS_ROOT>\python.exe' -m py_compile '<WORK_ROOT>\generated.py'
if ($LASTEXITCODE -ne 0) { throw ('py_compile failed: {0}' -f $LASTEXITCODE) }

& '<TOOLS_ROOT>\python.exe' '<WORK_ROOT>\generated.py' '<WORK_ROOT>\job-config.json'
$exitCode = $LASTEXITCODE
```

- `python -m py_compile <file>` exits nonzero when any file fails to compile - a cheap syntax gate with no side effects beyond `__pycache__`.
- Python exit codes: 1 for uncaught exceptions, 2 for usage errors. Judge by exit code, not by stderr presence.
- For console-encoding issues on Windows, `PYTHONIOENCODING` or `python -X utf8` are the controls to use.

## Encoding both directions

- PowerShell 5.1 `Set-Content -Encoding UTF8` writes a BOM. Python source files tolerate a UTF-8 BOM, and readers should use `utf-8-sig` for data files PowerShell wrote.
- Files Python writes with plain `utf-8` have no BOM; PowerShell reads them fine with `Get-Content`.
- Use the BOM-less .NET writer (see `files-paths-and-encoding.md`) when a consumer rejects BOM.

## Artifact routing and cleanup

Generated Python, its inputs, its logs, and `__pycache__` side effects belong under `<WORK_ROOT>` (or the task-assigned output root) - never inside `.agents\skills` or source folders. Long-running generated scripts should print a START line, phase markers, paths, periodic progress, and a final deterministic status.
