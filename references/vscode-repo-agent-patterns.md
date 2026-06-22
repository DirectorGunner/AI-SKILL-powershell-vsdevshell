# VS Code Repository Agent Patterns

Start by reading the repository's local agent instructions. They define allowed paths, artifact routing, validation expectations, and forbidden actions, and they always win over generic guidance.

## Detecting repo conventions

Look for, in rough priority order:

- Agent instruction files: `CLAUDE.md`, `AGENTS.md`, or equivalents at the repo root (and nested ones for subfolders).
- Contributor docs: `CONTRIBUTING.md`, docs folders describing build/test workflow.
- VS Code workspace config: `.vscode/tasks.json` (task `type` `shell` runs through a shell; `process` launches the program directly - official: VS Code tasks docs), `.vscode/settings.json`, `*.code-workspace`.
- Build manifests that reveal the toolchain: `*.sln`/`*.vcxproj` (MSBuild), `Cargo.toml` (Rust), `package.json` (npm scripts run via `cmd.exe` on Windows), `requirements.txt`/`pyproject.toml` (Python).

Derive the artifact roots from those instructions and map them to the placeholders below. If no convention exists, ask or propose one; do not scatter artifacts.

## Placeholder conventions for portable docs

Reusable documentation uses placeholders, never machine-specific paths:

- `<REPO_ROOT>` - the repository root.
- `<WORK_ROOT>` - scratch scripts, temp payloads, work logs.
- `<REPORTS_ROOT>` - user-facing reports and audit artifacts.
- `<PROMPTS_ROOT>` - prompt archives.
- `<TOOLS_ROOT>` - external tool installs.
- `<VSINSTALLDIR>` - the resolved Visual Studio instance root.

A skill consumer substitutes the current repo's real roots at use time.

## Artifact routing

- Prompt archives -> `<PROMPTS_ROOT>`; scratch/work/logs -> `<WORK_ROOT>`; reports -> `<REPORTS_ROOT>`.
- NEVER place task scripts, logs, reports, prompt archives, source captures, downloaded docs, or temp files inside `.agents\skills`. Skill folders hold reusable skill content only (SKILL.md plus references).
- Timestamped flat file names (for example `yyyyMMdd-HHmmss-description.ext`) keep artifact roots auditable (practical tip).

## Portability rule for skills

Skill documentation must stay reusable across repositories: no single project's internal implementation details, pass history, or machine paths. Record machine-specific observations only as clearly labeled local evidence (for example in a source map), never as universal assumptions in topic guidance.

## Source hygiene

- Do not edit public README files or installer scripts unless the task explicitly asks.
- Keep source changes narrow and evidence-backed; separate implementation files from documentation artifacts.
- Do not create hardlinks or symlinks unless explicitly authorized.
- Do not run Git commands unless the task explicitly requests Git validation; when skipped on request, report exactly that it was skipped by user preference.

## Maintaining this skill's metadata

When adding or materially changing guidance in any topic file: update `topics.json` (warnings, applies_to) and add corresponding retrieval chunks to `corpus.jsonl`. Keep the metadata in sync with the topic files.

## Reporting

Reports list changed files, sources inspected, validations run with exit codes, skipped checks with reasons, and remaining blockers. Use deterministic status labels (`VALIDATED`, `FAILED_WITH_EVIDENCE`, `SKIPPED_BY_SCOPE`, `BLOCKED_WITH_EVIDENCE`, `NOT_ATTEMPTED`) instead of vague confidence wording.
