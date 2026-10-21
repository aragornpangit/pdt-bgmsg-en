# Bootstrapping the group (DM / TM / rm sessions)

PM reads this reference only at startup: probe the terminal, detect partner sessions, create sessions, assign roles. PM is the group entry point, it does not open new sessions for itself, it only tops up `dm` / `tm`; `rm` is started on demand (see section 4).

## 1. Probe the terminal emulator

Probe it yourself first, do not ask the user right away:

```bash
env | grep -E '^(HERDR_ENV|HERDR_PANE_ID|TMUX|TMUX_PANE|WEZTERM_PANE|WEZTERM_UNIX_SOCKET|WT_SESSION|GHOSTTY_RESOURCES_DIR|TERM_PROGRAM|WSL_DISTRO_NAME)='
```

Take the **first hit** in the order of the table below as the host terminal (sessions can nest: WSL inside Windows Terminal, tmux inside WSL; take the innermost one you can control):

| Matched variable | Terminal | Can list sessions | How to create |
|---|---|---|---|
| `HERDR_ENV=1` | Herdr | yes, `herdr tab list` | herdr skill |
| `TMUX` | tmux | yes, `tmux ls` | `tmux new-session` |
| `WEZTERM_PANE` | WezTerm | yes, `wezterm cli list` | `wezterm cli spawn` |
| `WT_SESSION` | Windows Terminal | no | `wt.exe new-tab` |
| `GHOSTTY_RESOURCES_DIR` or `TERM_PROGRAM=ghostty` | Ghostty | no | open a new window |
| `WSL_DISTRO_NAME` | WSL (no multiplexer) | no | open a new window / new tab |

**Tell the user** the probe result ("detected host terminal is X, partner sessions will be created with it"), and write it into the `terminal` field of `.pdt/team.json`, then reuse it from then on.

Ask the user in only two cases: no variable matched at all; or the detected terminal does not match reality (the user can override by editing the `terminal` field in `.pdt/team.json`). When asking, use AskUserQuestion with the options: Herdr / tmux / WezTerm / other (Windows Terminal, Ghostty, WSL and other terminals that cannot list sessions).

## 2. Registry file

`.pdt/team.json` (project root, `mkdir -p` if the directory is missing):

```json
{
  "terminal": "tmux",
  "agent_cmd": "<command line that starts the agent>",
  "sessions": {
    "dm": { "id": "dm", "created": "<ISO time>", "cwd": "/home/user/proj" },
    "tm": { "id": "tm", "created": "<ISO time>", "cwd": "/home/user/proj" },
    "rm": { "id": "rm", "created": "<ISO time>", "cwd": "/home/user/proj" }
  }
}
```

- `terminal`: probe result or user choice, do not ask again once written
- `agent_cmd`: the command line that starts the partner agent, defaults to the same agent PM itself uses; if different, ask the user once
- `sessions`: records only partner sessions created by PM (not PM itself), label set `{dm, tm, rm}`; `rm` is created on demand, do not write this key if it was never started. In the same round, create the **project-root inbox skeleton** (a runtime channel, not committed): `mkdir -p inbox/{pm,dm,tm,rm}`, and confirm `.gitignore` contains `inbox/`. See the "Communication" section of [`group-conventions.md`](group-conventions.md). For listable terminals the live command is authoritative; for non-listable terminals this record is a clue only, not proof of liveness

## 3. Roll call: detect whether dm / tm are running

Session names are fixed as `dm`, `tm`, with consistent case.

| Terminal | Detection |
|---|---|
| Herdr | `herdr tab list` to read the tab label list (**use this to locate a partner before sending a message**); `herdr agent list` to see whether the agent is ready |
| tmux | look for `^dm:`, `^tm:` in the `tmux ls` output |
| WezTerm | `wezterm cli list --format json`, look it up by pane title |
| Other | check `sessions` in `.pdt/team.json`; if still unsure, ask the user "are the dm / tm sessions already running?", do not blindly create duplicates |

## 4. Top up: create the missing sessions

First judge whether the terminal can be automated, the two paths are entirely different:

| Capability | Terminal | Approach |
|---|---|---|
| Can create from the command line and inject input | Herdr, tmux, WezTerm | PM starts them itself, see the per-terminal commands below |
| Can only open a window, cannot inject input | Windows Terminal, Ghostty, WSL new window | **hand to the user**: print the prompt for the matching role from "Manual-mode prompts" below, ask the user to open a new tab or window and paste it, wait for the user to confirm readiness |

In manual mode herdr direct connection is still primary; fall back to a one-line entry in `docs/status.md` only when herdr is unavailable (see "Status board" in [`group-conventions.md`](group-conventions.md)). PM cannot verify session liveness; take the user's confirmation plus the herdr messages reported by that role as authoritative.

### Herdr

```bash
test "${HERDR_ENV:-}" = 1
herdr tab create --label dm --cwd "$PWD" --no-focus   # read .result.root_pane from the JSON
herdr agent start dm --kind <harness kind> --pane <pane-id>   # look up the kind in the herdr skill, never from memory
herdr agent prompt dm "<role assignment message>"
```

The tab `--label` is the role name (`herdr tab list` locates the partner by it). `tm` the same way, `--label tm`. For CLI syntax, lifecycle states and reading IDs see the herdr skill.

**`rm` is started on demand** (only when a research-type request comes in): same way, `herdr tab create --label rm --cwd "$PWD" --no-focus`, but the user starts their own harness, **do not run `agent start` in it**. After starting, register the label in `.pdt/team.json`, ask the user to start their harness in that tab, and dispatch work only after they confirm it is ready. Do not send messages and do not retry repeatedly before the harness is up.

**How to message a partner**: locating the label -> pane ID -> `herdr agent prompt` is the full three-step procedure, together with the fallback channel and the token-saving rules, **whose only full text is in the "Communication" section of [`group-conventions.md`](group-conventions.md)**; this file does not restate it.

### tmux

```bash
tmux new-session -d -s dm -c "$PWD" "$AGENT_CMD"
tmux send-keys -t dm "<role assignment message>" Enter
```

`tm` the same way.

### WezTerm

```bash
wezterm cli spawn --cwd "$PWD" -- "$AGENT_CMD"          # returns the new pane id
wezterm cli send-text --pane-id <id> "<role assignment message>"$'\n'
```

### Terminals that cannot receive injected input (Windows Terminal / Ghostty / WSL)

Try opening the window first, but you still cannot deliver the prompt after it opens:

```powershell
wt.exe -w 0 new-tab --title dm -d <project path> <agent_cmd>      # Windows Terminal
```

```bash
nohup ghostty --working-directory "$PWD" &                     # Ghostty: run $AGENT_CMD yourself inside the window
```

Opening the window only saves the user one step, **the prompt still has to be pasted by the user**. Go straight to the "Manual-mode prompts" flow below, do not automate for the sake of automating.

## 5. Role assignment message (auto-injected version)

For terminals that can be automated, inject this short message; the full pasteable prompts for manual mode are in section 7. A new session cannot see this session's context, so the message must be self-contained:

DM version:

```
You are DM (development manager, head of the dev development department). Read the skill file ~/.agents/skills/dm/SKILL.md in full before doing anything.
Project root: <cwd>.
Start action: wait for pm to dispatch; once received, set design decisions per the skill workflow and write docs/dev/plan/{spec-seq}-devPlan-{semantic-slug}-{YYMMDDHHMM}-v1.md yourself (50-line baseline, seq reuses the SPEC number).
Communication: direct connection with pm / tm, delivered per the "Communication" section of group-conventions (body goes to inbox/dm/, scan `ls -A inbox/dm/` at the start of every round, delete once read, no inbox mail receipt); no status documents, no document every round.
Report to pm once ready: session name, cwd, ready state. When blocked, include file:line or command output.
```

TM version:

```
You are TM (test manager, head of the test test department). Read the skill file ~/.agents/skills/tm/SKILL.md in full before doing anything.
Project root: <cwd>.
Start action: wait for pm to dispatch; once received, read the latest SPEC under docs/pd/spec/ and the latest plan under docs/dev/plan/ ({spec-seq}-devPlan-*.md), design cases per the skill workflow, and write docs/test/report/{spec-seq}-testReport-{semantic-slug}-{YYMMDDHHMM}-v1.md yourself (40-line baseline, cases and results in the same table); derive n parallel te per the actual situation.
Communication: direct connection with pm / dm, delivered per the "Communication" section of group-conventions (body goes to inbox/tm/, scan `ls -A inbox/tm/` at the start of every round, delete once read, no inbox mail receipt); no status documents, no document every round.
Report to pm once ready: session name, cwd, ready state. When blocked, include file:line or command output.
```

## 6. Discipline

- After starting, first confirm the session is really running (Herdr: `herdr agent get <name>`; tmux: `tmux ls`; WezTerm: `wezterm cli list`), then move on to requirement work
- If creation fails, or the user does not want PM to operate their terminal automatically, hand the commands to the user verbatim to run manually, and continue only after the user confirms the session is ready
- Roll call the PM session once per round: partners already online are not recreated and are not sent the role assignment again
- That PM creates the partner sessions is written into `.pdt/team.json`; re-probe when the terminal or the project changes

## 7. Manual-mode prompts (for pasting)

When the terminal cannot be automated, ask the user to open two new tabs or windows and paste the matching role prompt below **in full**, one prompt per session only. Before pasting, PM first replaces `{project root}` with the actual absolute path, leave the rest as is.

> **Maintenance note (isomorphic pair)**: The DM / TM sections below are **deliberately isomorphic**, only "your output" and "your subagents" differ by role. This is not a copy-paste accident, do not "deduplicate": both sections must be pasteable independently into a fresh session. When changing the **shared clauses** (communication, document discipline, wrap-up discipline) of one section, **both sections must change together**.

### DM

```
You are DM (development manager, head of the dev development department). Read the skill file ~/.agents/skills/dm/SKILL.md in full before doing anything.

Project root: {project root}
Your input: the latest SPEC under docs/pd/spec/; for fix-type work, the latest report under docs/test/report/
Your output: docs/dev/plan/{spec-seq}-devPlan-{semantic-slug}-{YYMMDDHHMM}-v1.md (development plan and progress, update in place within the version, 50-line baseline, you write it yourself; seq reuses the SPEC number)
Your subagents: de x n (n = number of parallelisable tickets, derive per the actual situation), they report to you only

Communication (direct connection between pm / dm / tm, managers talk to each other directly; delivery per the "Communication" section of group-conventions, body goes to the inbox file):
- Progress, acceptance conclusions, defect returns, arbitration requests are all delivered there, no status documents
- Do not write documents every round: update only when the plan changes substantially
- No polling: upstream updates are triggered by the user / pm, or checked once at the start of the round's task

Now: read the latest SPEC under docs/pd/spec/, set design decisions and the development plan per the skill workflow. When done, report back: session role, project root, SPEC number read, path of the file produced.
```

### TM

```
You are TM (test manager, head of the test test department). Read the skill file ~/.agents/skills/tm/SKILL.md in full before doing anything.

Project root: {project root}
Your input: the latest SPEC under docs/pd/spec/, the latest plan under docs/dev/plan/ ({spec-seq}-devPlan-*.md)
Your output: docs/test/report/{spec-seq}-testReport-{semantic-slug}-{YYMMDDHHMM}-v1.md (test report, cases and results in the same table, 40-line baseline, you write it yourself; seq reuses the SPEC number)
Your subagents: te x n (n = number of parallelisable batches, derive per the actual situation), they report to you only

Communication (direct connection between pm / dm / tm, managers talk to each other directly; delivery per the "Communication" section of group-conventions, body goes to the inbox file):
- Test conclusions, defect returns, arbitration requests are all delivered there, no status documents
- Every conclusion must carry file:line or grep evidence, conclusions without evidence are not accepted
- Do not write documents every round: write to disk only when the report is finalised or cases are appended
- No polling: upstream updates are triggered by the user / pm, or checked once at the start of the round's task

Now: read the latest SPEC and development plan, design cases per the skill workflow and dispatch te to execute them. When done, report back: session role, project root, SPEC number read, path of the file produced.
```

### Process and discipline

1. PM detects the host terminal cannot be automated -> print this to the user:
   > The current terminal ({terminal name}) cannot create sessions from the command line. Please open two new tabs or windows and paste the DM / TM prompt blocks above into them in full. Let me know once both are up.
2. After the user opens them and replies, PM writes both sessions into `.pdt/team.json`, marked `"managed": "manual"`
3. Before the user opens them, PM does not enter requirement work, wait for confirmation first
4. Afterwards the group communicates mainly via directed herdr messages; fall back to a one-line entry in `docs/status.md` only when herdr is unavailable
