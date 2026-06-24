---
name: memanto-pi-setup
description: Use when installing Memanto, enabling Memanto-backed persistent memory for pi projects, or adding the reusable Memanto/pi setup skill to a sksync bundle.
---

# Memanto pi Setup

## Overview

Memanto gives pi projects persistent memory through shell commands. There is no upstream `memanto connect pi` command yet, so the practical project-level integration is to use Memanto's Codex integration because it writes `AGENTS.md` and `.agents/skills/`, both of which pi can load.

Separate the work into two phases:

1. Machine setup: install and configure `memanto` once.
2. Project setup: activate a per-project Memanto agent and install project instructions/skills.

## Hard Rules

- Do not invent or run `memanto connect pi`.
- Do not store secrets, API keys, tokens, credentials, private customer data, or large unreviewed dumps in Memanto.
- Treat all `memanto` operations as shell commands; run them with the terminal/Bash tool.
- Prefer `.agents/skills/` for project-level compatibility with pi and other Agent Skills runtimes.
- For `remember`, always pass `--type`, `--confidence`, `--provenance`, `--source pi`, and useful `--tags`.

## Machine Setup

```bash
python3 --version
python3 -m pip install --upgrade memanto
command -v memanto
memanto
```

If Python reports an externally managed environment, use a virtualenv or `pipx install memanto` instead of forcing a system install.

During `memanto`, choose one backend:

| Backend | Use when | Notes |
|---|---|---|
| Cloud | Fastest setup | Needs Moorcheh API key from `https://console.moorcheh.ai/api-keys`; never commit it. |
| On-Prem | Local-only preference | Needs Docker; setup may pull Moorcheh/Ollama resources. |

Check configuration:

```bash
memanto config show
memanto config backend
```

## Apply to a pi Project

Run from the target project root.

### 1. Pick a stable agent id

Use a project slug so memory stays scoped:

```bash
PROJECT_SLUG=$(basename "$PWD" | tr '[:upper:]' '[:lower:]' | tr -cs 'a-z0-9-' '-')
AGENT_ID="pi-${PROJECT_SLUG%-}"
echo "$AGENT_ID"
```

### 2. Create or activate the agent

```bash
memanto agent list
memanto agent create "$AGENT_ID" --pattern tool --description "pi memory for $PROJECT_SLUG" \
  || memanto agent activate "$AGENT_ID"
```

`agent create` also activates a session. If the session expires later, run `memanto agent activate "$AGENT_ID"`.

### 3. Install project integration for pi

Use Codex integration as the pi-compatible path:

```bash
memanto connect codex --project-dir .
```

Expected files:

```txt
AGENTS.md
.agents/skills/memanto/SKILL.md
```

Verify the project application:

```bash
test -f AGENTS.md
test -f .agents/skills/memanto/SKILL.md
grep -n "MEMANTO-MANAGED-SECTION" AGENTS.md
```

Pi loads `AGENTS.md` from the project/parents and loads project `.agents/skills/` after project trust. Restart pi after applying, or explicitly launch pi with `--skill .agents/skills/memanto/SKILL.md` for a one-off check.

If `AGENTS.md` already has a Memanto block, do not duplicate it. The upstream connect command uses Memanto sentinel markers; prefer re-running it over manual copy/paste.

### 4. Warm up memory at session start

```bash
memanto agent activate "$AGENT_ID"
memanto recall "instructions preferences decisions goals" --limit 20
memanto answer "What should I know before working on this project?"
```

Use `recall` when you need raw memory chunks to guide work. Use `answer` when the user asks a direct question like “what did we decide?” or “what do I prefer?”.

## Remembering Conventions

Good memories are durable and actionable. Ask: “Will this matter next week?”

```bash
memanto remember "Decision: Use Supabase Auth for this project because it matches the existing stack." \
  --type decision \
  --confidence 0.95 \
  --provenance inferred \
  --source pi \
  --tags "auth,architecture"
```

Common types:

| Type | Store |
|---|---|
| `preference` | User/team preferences. |
| `instruction` | Standing rules that should affect future sessions. |
| `decision` | Architecture or implementation decisions and rationale. |
| `learning` | Corrections and lessons from mistakes. |
| `error` | Error symptoms plus verified fix. |
| `goal` | Project goals or milestones. |
| `context` | Session/project summaries. |

Provenance values: `explicit_statement`, `inferred`, `observed`, `corrected`, `validated`, `imported`.

Confidence guide: `1.0` explicit user instruction; `0.9-0.95` verified decision/fact; `0.8-0.85` repeated observed pattern; below `0.6` do not store.

## Add This Skill to the sksync Bundle

In the skills repository, keep the skill at:

```txt
skills/memanto-pi-setup/SKILL.md
```

Add to `sksync.config.json` dependencies:

```json
"memanto-pi-setup": {
  "agents": ["antigravity", "claude-code", "codex", "gemini-cli", "hermes", "jcode", "opencode", "pi"],
  "source": "./skills/memanto-pi-setup"
}
```

Add to `bundle/sksync.bundle.json` entries:

```json
"memanto-pi-setup": {
  "source": "../skills/memanto-pi-setup"
}
```

Validate with repository-specific sksync commands if available. At minimum, inspect both JSON files for valid syntax.

## Common Mistakes

| Mistake | Fix |
|---|---|
| Looking for `memanto connect pi` | Use `memanto connect codex --project-dir .`. |
| Using one global agent for every project | Use `pi-<project-slug>` per project. |
| Forgetting activation | Run `memanto agent activate <agent-id>` at session start. |
| Storing vague notes | Include concrete subject, rationale, refs, and tags. |
| Storing secrets | Never store secrets; store only non-sensitive decisions or instructions. |
| Duplicating `AGENTS.md` blocks | Re-run `memanto connect codex` and rely on sentinel markers. |
