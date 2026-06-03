# herdr-delegation verification

Date: 2026-06-01

## First verification

Two agents read `skills/herdr-delegation/SKILL.md` and applied it to scenarios.

Passes:

- Used `HERDR_ENV` hard gate.
- Used `herdr pane list` and focused pane as parent.
- Created separate workspace.
- Spawned labeled children with `herdr agent start ... -- pi "<prompt>"`.
- Injected `pane send-text` + `send-keys Enter` report convention.
- Re-listed panes before rereport/steering.
- Closed workspace and verified panes/processes.

Ambiguities found:

- Parent pane extraction from `herdr pane list` was described but not scripted.
- Workspace ID could be lost; skill needed explicit save/recovery guidance.
- Cleanup process grep could match parent `pi`; needed warning to filter by child labels.

## Refactor

Updated skill to:

- Include a `python3` one-liner to extract focused parent pane.
- Save workspace ID to `/tmp/herdr-child-workspace` and mention recovering via child labels.
- Clarify `workspace close` terminates panes in that workspace.
- Prefer child-label process grep over broad `pi` grep.

## Re-verification

A follow-up agent read the updated skill and reported no remaining blocking issues for parent discovery, workspace ID handling, rereport, or cleanup verification.

## Issue #4 verification

Date: 2026-06-03

Scenario: parent is inside herdr coordinating a dependent issue chain `MIK-102 -> MIK-103 -> MIK-104` with implementer/reviewer child agents in one child workspace; child pane IDs can compact/change; a reviewer may report `BLOCK`; child agents should not create/merge PRs unless explicitly delegated.

A fresh-context verification agent read the updated `skills/herdr-delegation/SKILL.md` and `tests/herdr-delegation/baseline.md`.

Passes:

- Requires re-resolving child panes by stable label before every `pane read`, `pane send-text`, or cleanup action.
- Defines a reviewer `BLOCK -> implementer fix -> re-review` loop and blocks progress until `APPROVE`.
- Requires sequential dependent issue waves: one issue at a time, finish/merge/pull current approved PR before the next dependent issue.
- States children must not create or merge PRs unless explicitly delegated; parent owns PR creation/merge by default.

Remaining ambiguity: none blocking.

Final verification result: PASS for GitHub issue #4 acceptance criteria.
