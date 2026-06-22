# MSBuild and MSVC Build Tools

Run MSBuild and MSVC from a verified Visual Studio Developer Shell environment unless the task provides fully resolved tool paths and environment variables. Developer Shell setup belongs at the TOP of a build script, once, followed by verification - not sprinkled mid-script and never switched within one shell.

## MSBuild command-line patterns

Every switch has `-switch` and `/switch` forms; switches are case-insensitive.

```powershell
$msbuildArgs = @(
    '<REPO_ROOT>\Project.sln',
    '-restore',                    # run Restore target first (NuGet)
    '-m',                          # parallel: without the switch the default is 1 process
    '-t:Build',                    # targets; semicolon-separate multiples
    '-p:Configuration=Release',
    '-p:Platform=x64',
    '-verbosity:minimal',
    '-bl:<WORK_ROOT>\build.binlog' # binary log for post-mortem analysis
)
& msbuild @msbuildArgs
$exitCode = $LASTEXITCODE
if ($exitCode -ne 0) { throw ('MSBuild failed with exit code {0}' -f $exitCode) }
```

- `-p:Name=Value` sets project properties; quote the whole element if a value has spaces.
- From PowerShell (a shell other than cmd), semicolon/comma lists for a switch may need quotes so the shell does not split them.
- `-targets` (list targets) and `-getProperty:` are non-building diagnostics.
- Since VS2019 16.5, MSBuild does not take toolset/library selection from the command-line environment; project files and properties control that. The Developer Shell still controls which `msbuild.exe`/tools are on PATH.

## MSVC environment verification

x86/x64 host-and-target combinations come from the environment that `VsDevCmd.bat`/`vcvarsall.bat`/`Enter-VsDevShell` set up (`vcvarsall.bat amd64`, `amd64_x86`, etc.). Verify before compiling and verify the ARCHITECTURE matches intent:

```powershell
& cl.exe /Bv          # compiler banner + version; nonzero/missing means environment problem
$clExit = $LASTEXITCODE
if ([string]::IsNullOrWhiteSpace($Env:INCLUDE) -or [string]::IsNullOrWhiteSpace($Env:LIB)) {
    throw 'INCLUDE/LIB are empty: the MSVC environment is not loaded.'
}
if ($Env:VSCMD_ARG_TGT_ARCH -ne 'x64') { throw ('Wrong target arch: {0}' -f $Env:VSCMD_ARG_TGT_ARCH) }
```

If the compiler cannot find standard headers or libraries, fix the environment (reload the correct Developer Shell) rather than hardcoding include/lib paths, unless the build system explicitly requires them.

## Build reporting

Record in the task log:

- Developer environment evidence (`VSCMD_VER`, arch variables, `VCToolsInstallDir`, `WindowsSdkDir`).
- Selected tool paths (which `msbuild.exe`, which instance).
- Full command argument lists and exit codes.
- Output/binary log paths.

Trust exit codes over partial output files: a leftover binary from an earlier build is not success evidence when MSBuild, `cl.exe`, or `link.exe` returned nonzero.

## See also

- [Visual Studio Developer Shell](visual-studio-devshell.md)
