# memanto-pi-setup verification

Date: 2026-06-24

## First verification

A fresh-context verification agent read `skills/memanto-pi-setup/SKILL.md` and `tests/memanto-pi-setup/baseline.md`.

Verdict: PASS.

Passes:

- Correctly states there is no upstream `memanto connect pi` and forbids inventing/running it.
- Gives concrete machine setup/config checks: Python, pip install, `command -v memanto`, `memanto`, `memanto config show`, and `memanto config backend`.
- Forbids storing secrets/API keys/tokens/private data.
- Requires full `remember` metadata and useful tags.
- Separates target project setup from machine setup and creates a stable `pi-<project-slug>` agent id.
- Uses the pi-compatible project integration path: `memanto connect codex --project-dir .`, producing `AGENTS.md` and `.agents/skills/memanto/SKILL.md`.
- Warns against duplicate Memanto blocks and prefers sentinel-marker reruns.
- Distinguishes `recall` vs `answer`.
- Gives explicit `sksync.config.json` and `bundle/sksync.bundle.json` snippets.
- Actual repository state includes `memanto-pi-setup` in both sksync config and bundle.

Recommended refinements:

1. Add fallback guidance for externally managed Python environments.
2. Make create-or-activate shell example idempotent.
3. Add a concrete project-application verification checklist after `memanto connect codex`.

## Refactor

Updated `skills/memanto-pi-setup/SKILL.md` to:

- Mention virtualenv or `pipx install memanto` when system Python rejects direct pip installs.
- Use `memanto agent create ... || memanto agent activate ...`.
- Verify `AGENTS.md`, `.agents/skills/memanto/SKILL.md`, and `MEMANTO-MANAGED-SECTION` after `memanto connect codex`.
- Mention one-off pi verification with `--skill .agents/skills/memanto/SKILL.md`.

## Re-verification

A fresh-context re-verification agent read the updated skill and baseline.

Verdict: PASS.

Evidence:

- Externally-managed Python fallback handled by virtualenv/`pipx install memanto` note.
- Idempotent agent create-or-activate handled with `memanto agent create ... || memanto agent activate`.
- Project application verification handled after `memanto connect codex` with file checks and sentinel grep.
- sksync bundle inclusion documented in the skill and present in actual `sksync.config.json` and `bundle/sksync.bundle.json`.

Residual risk: verification is documentation/config verification only; Memanto commands were not executed against a real target project.

Final verification result: PASS.
