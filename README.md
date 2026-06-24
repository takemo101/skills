# takemo101/skills

Personal agent skills for takemo101.

## Skills

- [`herdr-delegation`](skills/herdr-delegation/SKILL.md) — conventions for using herdr as a lightweight child-agent runtime: spawn children in separate workspaces, inject parent report commands, steer/rereport, and clean up safely.
- [`memanto-pi-setup`](skills/memanto-pi-setup/SKILL.md) — install Memanto, apply it to pi projects via the Codex-compatible project integration, and include it in the sksync bundle.

## Authoring Rule

Skills in this repo should be written with documentation TDD:

1. Define pressure scenarios.
2. Run baseline agents without the skill.
3. Capture failures/rationalizations.
4. Write the skill to address those failures.
5. Re-run scenarios with the skill and revise.

See `tests/` for captured test notes.
