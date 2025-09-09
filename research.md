# Research: Codebase Overview and Cursor CLI Integration Considerations

## Repository Overview

- Name: `specify-cli` (Python, Typer-based CLI)
- Packaging: `pyproject.toml` with `hatchling`; console script entry `specify = "specify_cli:main"`
- Source: `src/specify_cli/__init__.py` contains the entire CLI implementation
- Docs: `docs/` (DocFX), repo `README.md` explains Spec-Driven Development and CLI usage
- Templates: `templates/` including `spec-template.md`, `plan-template.md`, `/templates/commands/{specify,plan,tasks}.md`
- Scripts: `scripts/` shell helpers orchestrating feature creation and plan/task setup
- Memory: `memory/constitution.md` and related checklist

## Current CLI Capabilities (Typer app `specify`)

- Commands:
  - `specify init [project_name] [--ai claude|gemini|copilot] [--ignore-agent-tools] [--no-git] [--here]`
    - Downloads a template zip from `github/spec-kit` releases by AI assistant
    - Extracts to project directory (flattening GitHub archive nesting)
    - Optionally initializes a new git repo and commits
    - Displays a rich progress tree (no emojis), ASCII banner, and next steps
  - `specify check`
    - Verifies internet connectivity and optionally the presence of `git`, `claude`, `gemini`

- UX:
  - Arrow-key selection via `readchar` and `rich` Live UI
  - Uses `httpx` for GitHub API and streaming download

- Dependencies: `typer`, `rich`, `httpx`, `platformdirs`, `readchar`

## Template and Process Flow

- `/templates/commands/specify.md` → instructs agents to run `scripts/create-new-feature.sh --json` and fill `templates/spec-template.md`
- `/templates/commands/plan.md` → instructs agents to run `scripts/setup-plan.sh --json` and then execute `templates/plan-template.md`
- `/templates/commands/tasks.md` → instructs agents to run `scripts/check-task-prerequisites.sh --json` then create `tasks.md` from `/templates/tasks-template.md`
- Shell scripts manage branch naming, feature paths, and updating agent context via `scripts/update-agent-context.sh`

## Agent Context Update Script

- `scripts/update-agent-context.sh`
  - Maintains agent-specific files: `CLAUDE.md`, `GEMINI.md`, `.github/copilot-instructions.md`
  - Extracts tech details from current feature `plan.md` and updates relevant file(s)
  - No existing handling for Cursor agent files/rules

## Cursor CLI Concepts (from docs)

- Cursor CLI provides terminal access to Cursor Agent with support for interactive/non-interactive sessions.
- Cursor Rules: repository-scoped rules live under `.cursor/rules/` and guide agent behavior; useful for consistent command execution and project conventions.
- Installation (typical): `curl https://cursor.com/install -fsS | bash`; binary available as `cursor-agent` only.

## Gaps to Add Cursor Support

1. No explicit `cursor` option in `AI_CHOICES` (currently: `copilot`, `claude`, `gemini`).
2. No template variant for Cursor inside release assets (assumed pattern `spec-kit-template-<ai>`). Need `spec-kit-template-cursor` or unified template with Cursor artifacts.
3. No Cursor agent context artifacts are produced in Phase 1 (currently creates `CLAUDE.md`, `GEMINI.md`, `.github/copilot-instructions.md`). Need Cursor equivalent:
   - `.cursor/rules/` directory with initial rule set aligned to project
   - Possibly `.cursor/config` or MCP configuration if applicable
4. `scripts/update-agent-context.sh` does not update Cursor artifacts; must extend to support updating `.cursor/rules` or a consolidated `CURSOR.md` guideline.
5. Documentation lacks Cursor setup and usage sections in `README.md` and `docs/`.

## Constraints and Integration Points

- Release Artifact Dependency:
  - `download_template_from_github()` filters assets by `spec-kit-template-<ai_assistant>`. Adding Cursor requires a matching release asset name or fallback logic.

- Non-interactive CI Usage:
  - `init` command is interactive when `--ai` not provided; ensure non-interactive flow works for Cursor via `--ai cursor`.

- Tool Availability Checks:
  - `check()` and `init` perform tool checks. For Cursor, add a check for `cursor-agent` and provide install hint.

- Rule Bootstrapping:
  - On `plan` Phase 1, agent-specific file creation is delegated to `update-agent-context.sh`. For Cursor, rules likely come from template; updates may be incremental (append or regenerate rules).

## Risks

- Template asset availability for Cursor; fallback path may be required until release asset exists.
- Rule format/versioning changes in Cursor docs; need to pin minimal viable rule structure.

## Initial Decisions (Proposed)

- Add `cursor` to `AI_CHOICES` and corresponding checks with an install hint to official docs.
- Expect template asset `spec-kit-template-cursor` to be available; if not, temporarily reuse a generic template and inject `.cursor/rules` during extraction.
- Extend `update-agent-context.sh` to manage `.cursor/rules/` content with minimal, merge-safe updates.
- Document Cursor usage in `README.md` and add a `docs/cursor.md` quickstart.

## Assumptions Clarified

- Cursor CLI binary is `cursor-agent` only (no `cursor` binary expected).
- `.cursor/rules/` is the correct rules directory recognized by Cursor.
- A release asset named `spec-kit-template-cursor` will be published; until then, CLI will inject a minimal rules scaffold as fallback.
- MCP/server configuration is out of scope for this phase.
- Rule bootstrap focuses on mapping `/specify`, `/plan`, `/tasks` to existing scripts with absolute paths; advanced rule authoring is deferred.

