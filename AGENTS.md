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

## Development Workflow

Develop in one skill-sized slice at a time.

1. Pick exactly one skill or documentation slice.
2. Confirm the scope and success criteria before editing.
3. Re-read the relevant existing skill and test notes before changing behavior, triggers, hard gates, commands, or conventions.
4. Add or update the failing baseline scenario first when the slice changes skill behavior.
5. Implement only that slice; avoid opportunistic edits to unrelated skills or tests.
6. Verify with a fresh-context subagent or equivalent pressure scenario.
7. Record results in `tests/<skill-name>/verification.md` before starting another skill slice.
8. Address review or verification feedback in the same slice.
9. Mark the slice complete only after the skill/documentation change, test notes, and review/verification are done.

Keep design details, long rationales, and one-off investigations in the relevant test notes or supporting docs. `AGENTS.md` should stay focused on reusable agent workflow.

## GitButler / but Workflow

Use the `but` GitButler workflow for all version-control work.

- Use `but status -fv` before version-control mutations.
- Use `but` instead of git write commands.
- Do not run `git add`, `git commit`, `git push`, `git checkout`, `git merge`, `git rebase`, or `git stash`.
- Add `--status-after` to `but` mutation commands when available.
- Use IDs reported by `but status -fv`, `but diff`, or `but show`; do not hardcode IDs.
- Create branches, commits, pushes, PR finalization, and merge steps through the GitButler flow.

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
