# Quoting and Native Commands

PowerShell parses arguments before a native executable ever sees them. PowerShell parsing is not `cmd.exe` parsing: a quoted command-line string is a string, not a command plus arguments.

## A command string is not a command

```powershell
$c = '"C:\Tools\tool.exe" --input "file.txt"'
& $c        # FAILS: the call operator does not parse strings into command + arguments
```

The call operator runs a command whose NAME is stored in a string or variable; it never splits a command line. Keep the executable path and the arguments separate.

## Call operator + argument arrays (the safe default)

```powershell
$exe  = '<TOOLS_ROOT>\tool with spaces.exe'
$argv = @('--input', '<WORK_ROOT>\input file.txt', '--verbose')
& $exe @argv
$exitCode = $LASTEXITCODE
if ($exitCode -ne 0) { throw ('tool failed with exit code {0}' -f $exitCode) }
```

Each array element is passed as one argument, including elements with spaces. Log the final argument list when debugging.

## Portable examples (placeholder paths)

```powershell
& '<TOOLS_ROOT>\python.exe' '<WORK_ROOT>\script.py' '--mode' 'validate'

$guiToolArgs = @('--background', '--python', '<WORK_ROOT>\job.py')   # Blender-like GUI tool in batch mode
& '<TOOLS_ROOT>\gui-tool.exe' @guiToolArgs

& cargo build --release --manifest-path '<REPO_ROOT>\Cargo.toml'     # exit status 101 on failure

& npm run build                                                       # npm runs scripts via cmd.exe on Windows; exit code propagates

$msbuildArgs = @('<REPO_ROOT>\Project.sln', '-m', '-t:Build', '-p:Configuration=Release', '-p:Platform=x64')
& msbuild @msbuildArgs                                                # inside a verified Developer Shell
```

MSBuild note: semicolon- or comma-separated switch lists may need quoting when invoked from PowerShell instead of `cmd.exe`.

## The stop-parsing token `--%`

When a native command needs literal characters that PowerShell would otherwise interpret, place `--%` before the arguments; everything after it is passed literally except `%NAME%` environment expansion:

```powershell
icacls X:\VMS --% /grant Dom\HVAdmin:(CI)(OI)F
```

Limits: Windows native commands only; effective until end of line or pipe; no stream redirection after it; no `$variables` after it. For arguments that must contain literal quote characters, the underlying ProcessStartInfo.Arguments escaping applies (`\"` or doubled quotes) - see about_Parsing, "Passing arguments that contain quote characters".

## `$LASTEXITCODE` vs `$?`

- `$LASTEXITCODE` is the exit code of the last native program or script run. Check it immediately after the native call and store it before anything else runs.
- `$?` is the success status of the last PowerShell command; for native commands it is True only when `$LASTEXITCODE` is 0. In 5.1, wrapping in `(...)`/`$(...)`/`@(...)` resets `$?` to True - do not read `$?` after grouping.
- Decide success from `$LASTEXITCODE` and the tool's documented contract, not from `$?` alone.

## stderr does not mean failure

Many tools write progress, warnings, or banners to stderr. Capture and log stderr, then judge failure by exit code:

```powershell
$outLog = '<WORK_ROOT>\tool.stdout.log'
$errLog = '<WORK_ROOT>\tool.stderr.log'
& $exe @argv 1> $outLog 2> $errLog
$exitCode = $LASTEXITCODE
```

Practical tip: when `$ErrorActionPreference = 'Stop'` is set, redirecting a native command's stderr inside PowerShell (for example `2>&1`) can wrap stderr lines in ErrorRecords (NativeCommandError) and make a successful command look failed. Prefer redirecting to files, or capture without merging, and always decide by exit code.

Use stdin redirection (for example piping empty input) only for known interactive blockers, and document why that tool needs it.
