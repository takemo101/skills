# herdr-delegation baseline tests

Date: 2026-06-01

Goal: capture what agents do without a dedicated herdr delegation skill.

## Scenario A — multi-child workspace

Prompt: agent is inside herdr and must create three child pi agents in a separate workspace, have them report to parent, and clean up.

Observed baseline behavior:

- Invented or assumed environment variables such as `HERDR_WORKSPACE_ID`, `HERDR_TAB_ID`, and `HERDR_PANE_ID`.
- Used complex manifest/script machinery before proving the simple `herdr agent start` flow.
- Used `pane split`/`pane run` directly rather than the simpler `herdr agent start <name> --workspace ... -- pi "<prompt>"` path we proved experimentally.
- Assumed `jq` and command response shapes.
- Used `pane run` for parent reporting, which can submit the report as terminal input; for pi we experimentally used `pane send-text` + `pane send-keys Enter`.
- Did not emphasize re-listing panes because herdr pane IDs can compact.

## Scenario B — one child and rereport

Prompt: spawn one child `pi`, have it report, later ask it to report again, then clean up.

Observed baseline behavior:

- Provided a child prompt but did not first show how to discover the focused parent pane with `herdr pane list`.
- Used placeholder `$PARENT_SESSION` / `$PARENT_PANE_ID` without showing how to bind them from live herdr output.
- Did not capture child label/name as the stable way to rediscover compacted pane IDs.
- Cleanup was only `pane close`, not workspace cleanup when a separate workspace was created.

## Scenario C — safety and quoting

Prompt: use herdr, not pi-subagents; put children in another workspace and report to parent.

Observed baseline behavior:

- Correctly warned about wrong-pane injection and pane ID compaction.
- Still suggested commands not verified in the local herdr version (`session list --json`, some env vars, direct `pane split` flow).
- Over-focused on shell quoting; simpler convention is to embed a literal report command in the child prompt using known parent pane ID and child label.

## Skill implications

The skill must teach:

1. Verify `HERDR_ENV=1` and stop if not inside herdr.
2. Discover parent pane from `herdr pane list` focused pane, not assumed env vars.
3. Create a separate workspace with `herdr workspace create --cwd ... --label ... --no-focus`.
4. Spawn children with `herdr agent start <name> --workspace <ws> --no-focus -- pi "<prompt>"`.
5. Include an explicit report convention in every child prompt:
   - `herdr pane send-text <parent_pane> '【<child> report】<summary>'`
   - `herdr pane send-keys <parent_pane> Enter`
6. For rereport/steer, re-run `herdr pane list`, locate by label, then `pane send-text` + Enter.
7. Clean up with `herdr workspace close <workspace_id>` for grouped child work.
8. Verify cleanup with `herdr pane list` and optionally `ps`.
