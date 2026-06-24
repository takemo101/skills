# memanto-pi-setup baseline tests

Date: 2026-06-24

Goal: capture what agents do without a dedicated Memanto-for-pi setup skill.

## Scenario A — create a reusable Memanto/pi setup skill

Prompt: in this skills repository, explain how to create a reusable skill/manual for installing Memanto and applying it to pi projects, then include it in the sksync bundle. Do not use a Memanto-specific skill.

Observed baseline behavior:

- Correctly identified the repository documentation-TDD workflow: create `tests/memanto-pi-setup/baseline.md`, then `skills/memanto-pi-setup/SKILL.md`, then `tests/memanto-pi-setup/verification.md`.
- Correctly found that `sksync.config.json` and `bundle/sksync.bundle.json` need new `memanto-pi-setup` entries.
- Suggested generic project setup steps but did not center the most practical pi path: `memanto connect codex --project-dir .` creates `AGENTS.md` plus `.agents/skills/`, both of which pi can load.
- Suggested `memanto memory sync --project-dir .` before establishing install, backend, active agent, or whether the command exists in the installed version.
- Noted a possible `sksync`/pi target-path mismatch (`.pi/agent/skills` vs pi docs' `.pi/skills` / `.agents/skills`) but did not turn that into a concrete guardrail.
- Did not give a concrete idempotent project-application checklist for avoiding duplicate `AGENTS.md` memory blocks.
- Did not specify safe memory content rules: do not store secrets, tokens, private credentials, or large unreviewed files.
- Did not define when to use `recall` vs `answer`, or require full metadata for `remember` calls.

## Scenario B — applying to an arbitrary pi project

Expected failure without the skill:

- Agent may try to create a pi-specific Memanto integration even though upstream Memanto has no `connect pi` command.
- Agent may install the skill only under a runtime-specific directory instead of using `.agents/skills/` for pi-compatible project-level loading.
- Agent may edit `AGENTS.md` manually without sentinel markers or duplicate-check guidance.
- Agent may skip checking whether `memanto` is already installed/configured and whether an active agent exists.
- Agent may confuse the reusable skill repo setup with per-project Memanto activation.

## Skill implications

The skill must teach:

1. Separate two jobs: installing/configuring Memanto on the machine, and applying Memanto conventions to a target pi project.
2. Prefer project application through `memanto connect codex --project-dir .` because pi reads `AGENTS.md` and `.agents/skills/`.
3. Use explicit preflight checks: Python/pip availability, `command -v memanto`, `memanto config show`, `memanto agent list`, and target project root.
4. Create or activate a stable per-project agent, typically `pi-<project-slug>`.
5. Use `memanto recall` for raw context and `memanto answer` for direct questions.
6. Require full metadata for `memanto remember`: `--type`, `--confidence`, `--provenance`, `--source pi`, and useful tags.
7. Avoid storing secrets or dumping sensitive/private files into memory.
8. Include the skill in both `sksync.config.json` dependencies and `bundle/sksync.bundle.json` entries.
9. Verify project application by checking generated files and, where possible, a fresh pi session or skill discovery path.
