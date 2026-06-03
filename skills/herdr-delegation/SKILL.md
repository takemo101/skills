---
name: herdr-delegation
description: Use when running inside herdr and delegating work to child AI agent panes or workspaces, especially when spawning, steering, rereporting, collecting reports, or cleaning up child agents.
---

# herdr Delegation

## Overview

Use herdr as the runtime, not as a workflow engine. The parent agent decides; child agents work in separate panes/workspaces and report back by sending text to the parent pane.

## Hard Gate

Before any herdr control:

```bash
test "${HERDR_ENV:-}" = 1 || { echo "Not running inside herdr"; exit 1; }
```

If not inside herdr, stop. Do not inspect or control focused panes from outside herdr.

## Core Pattern

### 1. Discover parent pane

```bash
PARENT_PANE=$(herdr pane list | python3 -c 'import sys,json; j=json.load(sys.stdin); print(next(p["pane_id"] for p in j["result"]["panes"] if p.get("focused")))')
echo "$PARENT_PANE"
```

Use the pane with `focused: true` as the parent. Save its `pane_id`. Do not assume `HERDR_PANE_ID`; pane IDs can compact after panes close.

### 2. Create a child workspace

```bash
WS_JSON=$(herdr workspace create --cwd "$PWD" --label "delegation-test" --no-focus)
WORKSPACE_ID=$(printf '%s' "$WS_JSON" | python3 -c 'import sys,json; j=json.load(sys.stdin); print(j["result"]["workspace"]["workspace_id"])')
printf '%s\n' "$WORKSPACE_ID" > /tmp/herdr-child-workspace
```

Save the workspace ID and child labels in your notes. If you lose the workspace ID, re-run `herdr pane list` and find the child labels; each pane includes `workspace_id`.

### 3. Start child agents

```bash
herdr agent start docs-scout \
  --cwd "$PWD" \
  --workspace "$WORKSPACE_ID" \
  --no-focus \
  -- pi "<child prompt>"
```

Put each child in the child workspace and give it a unique label (`docs-scout`, `store-scout`, etc.). Labels let you rediscover panes after ID compaction.

### 4. Always inject the report convention

Every child prompt must include the exact parent pane and report format:

```text
When done, report to parent pane <PARENT_PANE> with:

herdr pane send-text <PARENT_PANE> '【<child-name> report】<summary>'
herdr pane send-keys <PARENT_PANE> Enter
```

For long output: write a file and report the path. Do not paste huge transcripts into the parent pane.

### 5. Resolve child panes by label before every action

Treat child pane IDs as ephemeral. Before each child `pane read`, `pane send-text`, or cleanup action, re-run `herdr pane list` and resolve the current pane by stable label:

```bash
CHILD_LABEL="docs-scout"
CHILD_PANE=$(herdr pane list | CHILD_LABEL="$CHILD_LABEL" python3 -c 'import os,sys,json; j=json.load(sys.stdin); label=os.environ["CHILD_LABEL"]; print(next(p["pane_id"] for p in j["result"]["panes"] if p.get("label") == label))')
echo "$CHILD_PANE"
```

Keep labels unique per child role and issue (`mik-103-implementer`, `mik-103-reviewer`). Never reuse an old child pane ID after panes close, compact, or disappear.

### 6. Steer, read, or request rereport

Resolve by label first, then act on the current pane:

```bash
herdr pane send-text "$CHILD_PANE" "もう一度、親ペイン <PARENT_PANE> に【docs-scout report】形式で再報告してください。"
herdr pane send-keys "$CHILD_PANE" Enter
```

### 7. Clean up

For grouped child work, close the child workspace:

```bash
herdr workspace close <WORKSPACE_ID>
herdr pane list
```

Verify only intended parent panes remain. `workspace close` closes the child panes in that workspace; if a child is still running, it is terminated with the pane.

If needed, check processes by child labels first. A broad `pi` grep may match the parent, so interpret it carefully:

```bash
ps -axo pid,ppid,stat,command | grep -E 'docs-scout|store-scout|profile-scout' | grep -v grep || true
```

## Review and dependent-issue workflows

### Reviewer `BLOCK` loop

Use explicit reviewer outcomes for delegated implementation:

1. Implementer reports done with checks run and changed files/branch/PR draft state.
2. Reviewer reports `APPROVE` or `BLOCK: <blocker summary>`.
3. If `BLOCK`, parent sends the blocker summary back to the implementer pane.
4. Implementer fixes, reruns checks, and reports again.
5. Parent starts or reuses a reviewer/rereviewer pane.
6. Continue only after `APPROVE`.

No PR creation or merge while review is blocked.

### Sequential dependent issue wave

For chains like `MIK-102 -> MIK-103 -> MIK-104`:

- Use one shared child workspace for the wave.
- Use separate labels per issue and role (`mik-102-implementer`, `mik-102-reviewer`).
- Process exactly one dependent issue at a time.
- Finish implementation, review, fixes, approval, PR creation/merge, and latest pull for the current issue before starting the next dependent issue.
- Keep the parent as decision maker and PR owner.

### PR guardrails

Child agents must not create or merge PRs unless the parent explicitly delegates that action in the child prompt. By default, children implement, review, fix, run checks, and report. The parent reviews reports, creates/merges PRs, pulls latest, and decides when to start the next issue.

## Quick Reference

| Need | Command |
|---|---|
| Confirm inside herdr | `test "${HERDR_ENV:-}" = 1` |
| Find parent | `herdr pane list` → focused pane |
| New workspace | `herdr workspace create --cwd "$PWD" --label NAME --no-focus` |
| Spawn child | `herdr agent start NAME --workspace WS --no-focus -- pi "PROMPT"` |
| Report to parent | `pane send-text PARENT '【name report】...'` + `send-keys PARENT Enter` |
| Resolve child | `herdr pane list` → find current pane by stable label |
| Read/steer/rereport | Resolve label first, then `pane read` or `pane send-text CHILD ...` + Enter |
| Review block | Send `BLOCK` summary to implementer, fix, re-review; continue only on `APPROVE` |
| Dependent issue chain | One issue at a time; merge approved PR before starting next dependent issue |
| Cleanup group | Resolve/check labels, then `herdr workspace close WS` |

## Common Mistakes

| Mistake | Fix |
|---|---|
| Assuming parent pane env vars exist | Use `herdr pane list` and focused pane. |
| Using compacted stale pane IDs | Re-list panes and resolve by label before every `read`, `send-text`, or cleanup action. |
| Forgetting report instructions | Put exact `send-text` + `send-keys Enter` command in every child prompt. |
| Using herdr as scheduler | Parent decides; herdr only runs panes and carries messages. |
| Running dependent issues in parallel | Process one dependent issue at a time; finish approved PR merge before the next issue. |
| Treating reviewer `BLOCK` as advisory | Send blockers back to implementer and require re-review before PR creation/merge. |
| Letting children create or merge PRs by default | Parent owns PR creation/merge unless explicitly delegated. |
| Leaving child panes around | Close the child workspace and verify with `pane list`. |
| Pasting huge reports | Child writes files and reports paths. |

## Minimal Child Prompt Template

```text
You are <child-name>, a child agent running under herdr.

Task:
<task>

Rules:
- Keep work bounded.
- Do not edit files unless explicitly asked.
- Do not create or merge PRs unless explicitly asked.
- Write long findings to a file and report its path.
- When done, report to parent pane <PARENT_PANE>:

herdr pane send-text <PARENT_PANE> '【<child-name> report】<summary>'
herdr pane send-keys <PARENT_PANE> Enter
```
