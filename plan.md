# Plan: Add Support for Cursor CLI (Design Only)

This plan outlines the implementation steps to add first-class support for Cursor CLI and Cursor Rules. No code will be changed until approved.

## Goals

- Add `cursor` as a selectable AI assistant in `specify init`
- Provide a bootstrapped `.cursor/rules/` setup in generated projects
- Update agent context update flows to maintain Cursor artifacts
- Update docs and checks to cover Cursor installation and usage

## Scope

- CLI behavior additions in `src/specify_cli/__init__.py`
- Script updates in `scripts/update-agent-context.sh`
- Template and release asset conventions
- Documentation: `README.md` and `docs/`

## Assumptions

- Release assets support a new template named `spec-kit-template-cursor` (or a shared template can be adapted at extraction time).
- Cursor CLI binary is accessible as `cursor` and/or `cursor-agent`; we will check both.
- Cursor Rules live at `.cursor/rules` per docs.

## Implementation Steps

1) CLI: Add Cursor option and checks
- Extend `AI_CHOICES` to include `"cursor": "Cursor CLI"`.
- In `init` validation, accept `--ai cursor`.
- Tool checks:
  - Try `shutil.which("cursor")` OR `shutil.which("cursor-agent")`.
  - Install hint: `Install from: https://docs.cursor.com/en/cli/overview`.
- Adjust next steps messaging to include Cursor usage.

2) Template selection for Cursor
- In `download_template_from_github()` and `download_and_extract_template()`, the asset filter uses `spec-kit-template-{ai_assistant}`. Ensure `cursor` resolves to the correct asset.
- Fallback path (if asset not present):
  - Use a generic agent-agnostic template.
  - Inject a minimal `.cursor/rules/` scaffold during extraction (see Step 3).

3) Bootstrap `.cursor/rules/`
- Provide a minimal initial ruleset (files created during extraction if missing):
  - `.cursor/rules/00-repo-context.md`: overview and guardrails referencing `memory/constitution.md` and `/templates/commands/*` flows.
  - `.cursor/rules/10-commands-specify.md`, `10-commands-plan.md`, `10-commands-tasks.md`: small wrappers pointing Cursor Agent to run our scripts with absolute paths, mirroring existing command templates.
  - `.cursor/rules/90-safety.md`: guardrails for non-interactive commands and path safety (absolute paths, no destructive ops without explicit flags).
- Ensure idempotent creation: if rules already present (from template asset), do not overwrite; otherwise, create.

4) Agent context updater: add Cursor support
- Extend `scripts/update-agent-context.sh`:
  - Accept `cursor` as an `AGENT_TYPE`.
  - Manage a lightweight `CURSOR.md` (optional) OR update notes inside `.cursor/rules/00-repo-context.md` with tech entries derived from `plan.md` (language, dependencies, storage, project type).
  - Keep updates minimal and merge-safe (append-only within a delimited section or targeted substitutions).
- Summary output includes Cursor updates when applicable.

5) Docs updates
- `README.md`:
  - Add Cursor to prerequisites and quickstart examples: `specify init <name> --ai cursor`.
  - Add a short "Using with Cursor" section and link to official docs.
- `docs/`:
  - Add `docs/cursor.md`: installation, verifying `cursor --version`/`cursor-agent --version`, overview of `.cursor/rules/`, how /specify, /plan, /tasks map to rules.
  - Update `docs/quickstart.md` where we mention available agents.

6) `spec-kit` release alignment
- Coordinate GitHub Releases to include `spec-kit-template-cursor` asset with:
  - `.cursor/rules/` seeded
  - Existing `templates/` and `scripts/` included
- If coordination cannot happen immediately, keep the fallback injection in CLI (Step 2) and remove once asset is available.

7) Non-interactive and CI considerations
- Confirm `--ai cursor` path remains non-interactive.
- Ensure no prompts are shown when `--here` merges into non-empty repos unless explicitly confirmed (existing behavior uses `typer.confirm`). Document the flag and behavior for CI.

8) Safety and compatibility
- Limit file writes under `.cursor/` to idempotent operations.
- Maintain current behavior for other agents; avoid regressions.

## Testing Strategy

- Manual:
  - `specify check` shows Cursor tool check and hints when missing.
  - `specify init demo --ai cursor` downloads template and produces `.cursor/rules/`.
  - If `spec-kit-template-cursor` missing, verify fallback injection of rules.
  - Ensure `update-agent-context.sh cursor` updates Cursor artifacts without overwriting custom content.
- Automated (future optional):
  - Add smoke tests for extraction and rule injection in a temp dir.

## Rollout

- PR with feature behind code changes only; no template asset dependency enforced until release available.
- After merge, publish release with Cursor template asset; then remove/deprecate fallback injection if desired.

## Out of Scope (for now)

- MCP server configuration and advanced Cursor integrations.
- Deep rule authoring beyond minimal bootstrapping and guided commands.

## Approval Gate

- Proceed to implementation only after approval of this plan.