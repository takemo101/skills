# takemo101/skills Guidelines

This repository stores personal reusable agent skills.

## Repository Layout

```txt
skills/
  <skill-name>/
    SKILL.md

tests/
  <skill-name>/
    baseline.md
    verification.md
    regression-*.md   # optional future scenarios
```

## Iron Law

**No skill without a failing skill test first.**

Skill authoring in this repo follows documentation TDD:

1. Define pressure/application scenarios.
2. Run scenarios without the new or changed skill.
3. Record baseline failures/rationalizations in `tests/<skill-name>/baseline.md`.
4. Write or edit `skills/<skill-name>/SKILL.md`.
5. Run scenarios with the skill.
6. Record results in `tests/<skill-name>/verification.md`.
7. If agents find loopholes, update the skill and re-test.

This applies to new skills and edits to existing skills. Do not batch multiple unverified skills.

## Skill Names

- Use lowercase kebab-case.
- Use letters, numbers, and hyphens only.
- Prefer action or domain names that match search intent.
- Directory name, frontmatter `name`, and test directory should match.

Example:

```txt
skills/herdr-delegation/SKILL.md
tests/herdr-delegation/baseline.md
tests/herdr-delegation/verification.md
```

## SKILL.md Requirements

Frontmatter must include:

```yaml
---
name: skill-name
description: Use when ...
---
```

Description rules:

- Start with `Use when`.
- Describe triggering conditions only.
- Do not summarize the workflow.
- Keep it concise and searchable.

Body should usually include:

- Overview / core principle.
- When to use or hard gate, if relevant.
- Quick reference table.
- Minimal command or code examples.
- Common mistakes / failure modes.

Keep skills concise. Move long references or reusable scripts into supporting files only when necessary.

## Test Notes

`tests/<skill-name>/baseline.md` should capture:

- Scenario prompts.
- What agents did without the skill.
- Mistakes, missing checks, unsafe assumptions, rationalizations.
- Specific implications for the skill.

`tests/<skill-name>/verification.md` should capture:

- Scenarios run with the skill.
- What passed.
- Remaining ambiguities.
- Refactors made to SKILL.md.
- Final verification result.

These files are regression memory for future improvements.

## Editing Existing Skills

Before editing:

1. Read `skills/<skill-name>/SKILL.md`.
2. Read `tests/<skill-name>/baseline.md` and `verification.md`.
3. Add a new scenario if the requested change covers a new failure mode.
4. Test with a subagent or equivalent fresh context.
5. Update the test notes with what changed.

Do not make untested “simple wording fixes” if they affect behavior, triggers, hard gates, commands, or conventions.

## herdr-delegation Notes

The `herdr-delegation` skill was created after testing real herdr child-agent workflows.

Important conventions it must preserve:

- Stop if `HERDR_ENV != 1`.
- Discover parent pane from `herdr pane list` focused pane.
- Put children in a separate workspace.
- Use labeled `herdr agent start` children.
- Inject explicit `pane send-text` + `send-keys Enter` report commands into child prompts.
- Re-list panes before steering/rereporting.
- Close the child workspace and verify cleanup.

## Do Not

- Do not turn personal skills into project-specific narratives.
- Do not skip baseline testing because behavior seems obvious.
- Do not create multiple skills before verifying the current one.
- Do not put secrets, tokens, or private credentials in skills or test logs.
