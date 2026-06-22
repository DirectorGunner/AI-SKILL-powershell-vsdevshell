# Windows PowerShell 5.1 and Visual Studio Developer Shell Skill

> _Practical guidance for writing, generating, reviewing, and debugging Windows PowerShell 5.1 scripts and Visual Studio Developer Shell workflows in any Windows VS Code repository._

Part of **[Agent Kaizen](https://github.com/DirectorGunner/agent-kaizen)** — continuously improving how AI agents reason, act, verify, and operate inside project environments. This repository is the `powershell-vsdevshell` skill: a reusable, trigger-rich task handbook that an AI coding agent (OpenAI Codex, Claude Code) loads on demand when a task matches its triggers.

## What this skill covers

This skill targets Windows PowerShell 5.1 compatibility and Visual Studio Developer Shell workflows: writing 5.1-safe syntax (avoiding PS7-only ternary, `??`, `&&`/`||` chains, and `ForEach-Object -Parallel`), invoking native executables with the call operator, separate argument arrays, the `--%` stop-parsing token, and capturing `$LASTEXITCODE` rather than `$?`. It handles setup, build, and wrapper scripts that call Python, Cargo, npm, MSBuild, and MSVC, including loading and verifying the Developer Shell (`Launch-VsDevShell.ps1`, `vswhere` discovery, `VSCMD_VER`/`INCLUDE`/`LIB` checks). It also covers embedded Python generation, file/path/encoding handling (UTF-16 LE BOM defaults, BOM-less writers, long paths), stdout/stderr logging, arrays, hashtables, functions, artifact routing, and AST/JSON/exit-code validation before declaring success.

## What's inside

- `SKILL.md` — frontmatter (`name` + trigger-rich `description`) and a lean body.
- `references/` — right-sized topic files the agent loads only when relevant (plus `INDEX.md` and `topics.json`).
- `GOTCHA.md` — known pitfalls and edge cases.

## Use it

This skill is one git repo inside the Agent Kaizen **skills store**. The store nests two folders on purpose: the outer **`SKILLS\`** is a VS Code project for building and maintaining skills (its own workspace + tooling), and the inner lowercase **`skills\`** holds every skill as its own repo. That split lets a project pull skills two ways — the **whole `skills\` folder at once** (loads everything — **not recommended**) or **one skill at a time** (recommended: load only what a task needs and stay under Claude Code's skill-listing budget).

Paths below use **`%DEVROOT%`** — the `DEVROOT` environment variable pointing at the folder that contains `SKILLS\`. Set it once by running **`SetDevRoot.cmd`** in the SKILLS repo root (no admin); a new shell then resolves `%DEVROOT%` automatically.

Link **just this skill** (recommended) — a Windows directory junction, no admin:

```cmd
mklink /J .agents\skills\powershell-vsdevshell  "%DEVROOT%\SKILLS\skills\powershell-vsdevshell"
mklink /J .claude\skills\powershell-vsdevshell  "%DEVROOT%\SKILLS\skills\powershell-vsdevshell"
```

Or link the **whole store** at once (loads every skill — not recommended outside a skills-dev project):

```cmd
mklink /J .agents\skills  "%DEVROOT%\SKILLS\skills"
mklink /J .claude\skills  "%DEVROOT%\SKILLS\skills"
```

Remove a link (the store copy is untouched):

```cmd
rmdir .agents\skills\powershell-vsdevshell
rmdir .claude\skills\powershell-vsdevshell
```

The agent (OpenAI Codex, Claude Code) then auto-loads this skill whenever a task matches its triggers. Built and validated with **[skill-drafting](https://github.com/DirectorGunner/AI-skill-drafting)** to the Agent Kaizen gold standard.

## License

Licensed under **AGPL-3.0**, matching the [Agent Kaizen](https://github.com/DirectorGunner/agent-kaizen) project.
