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

### 5. Steer or request rereport

Re-list panes first, find the child by label, then send text:

```bash
herdr pane list
herdr pane send-text <CHILD_PANE> "もう一度、親ペイン <PARENT_PANE> に【docs-scout report】形式で再報告してください。"
herdr pane send-keys <CHILD_PANE> Enter
```

### 6. Clean up

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

## Quick Reference

| Need | Command |
|---|---|
| Confirm inside herdr | `test "${HERDR_ENV:-}" = 1` |
| Find parent | `herdr pane list` → focused pane |
| New workspace | `herdr workspace create --cwd "$PWD" --label NAME --no-focus` |
| Spawn child | `herdr agent start NAME --workspace WS --no-focus -- pi "PROMPT"` |
| Report to parent | `pane send-text PARENT '【name report】...'` + `send-keys PARENT Enter` |
| Rereport | `pane send-text CHILD "report again..."` + Enter |
| Cleanup group | `herdr workspace close WS` |

## Common Mistakes

| Mistake | Fix |
|---|---|
| Assuming parent pane env vars exist | Use `herdr pane list` and focused pane. |
| Using compacted stale pane IDs | Re-list panes before steering/cleanup; use labels. |
| Forgetting report instructions | Put exact `send-text` + `send-keys Enter` command in every child prompt. |
| Using herdr as scheduler | Parent decides; herdr only runs panes and carries messages. |
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
- Write long findings to a file and report its path.
- When done, report to parent pane <PARENT_PANE>:

herdr pane send-text <PARENT_PANE> '【<child-name> report】<summary>'
herdr pane send-keys <PARENT_PANE> Enter
```
