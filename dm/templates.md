# dev department templates (DM)

**One file per requirement**: `docs/dev/plan/{spec-seq}-devPlan-{semantic-slug}-{timestamp}-v{N}.md` (naming in [`../pm/group-conventions.md`](../pm/group-conventions.md) "File naming", seq reuses the SPEC number), containing design decisions, the ticket table and per-ticket progress. **Update in place within a version**, substantive revisions bump the version into a new file; no separate progress report, no separate ADR, no separate design document. **50-line baseline, moderate overflow allowed** (if over, delete redundancy first; if there is nothing left to delete, register the reason at the head of the file).

## Template

```markdown
# Development plan and progress: {feature title}

- Number: {spec-seq} | Version: v{N} (follows the SPEC) | Branch: `{branch}` (based on {hash})
- Related SPEC: `docs/pd/spec/{spec-seq}-spec-*.md` | Related test report: `docs/test/report/{spec-seq}-testReport-*.md`

## Design decisions

| # | Decision | Why | How |
|---|---|---|---|
| 1 | {one sentence} | {reason / constraint / tradeoff} | {file, function, constant; use seam / deep module vocabulary} |

## Ticket

| # | Goal (end-to-end behavior) | Blocked by | Files | TDD seam | Status |
|---|---|---|---|---|---|
| 01 | | None | | | |

## Per-ticket results

| Ticket | de | Commit | Build | Unit test | Acceptance | Status |
|---|---|---|---|---|---|---|

## Conclusion of this round

{within 3 lines: what got done, where it is stuck, next step. Runtime items pending verification go here}
```

**Sections may be deleted**: interface contracts, dependencies and batches, per-ticket results are deleted outright when the content is simple, do not leave empty tables. The ticket table is one table by default; only for complex tickets spanning >= 3 files or needing independent verification, expand a few lines below the table (goal, Blocked by, Files, acceptance criteria, TDD seam, definition of done). **Over the baseline, delete redundancy first; if there is nothing left to delete, register the reason at the head of the file**.

## Slicing details

Internalized from mattpocock [`to-tickets`](https://raw.githubusercontent.com/mattpocock/skills/main/skills/engineering/to-tickets/SKILL.md), DM executes it directly without relying on the `/to-tickets` command:

- Each slice cuts one **narrow but complete** path (through schema / API / UI / tests), **vertical rather than horizontally layered**; it can be demonstrated or verified independently; it can be completed within a single fresh context
- Do the **prefactor** first ("make the change easy first, then make the easy change")
- **Wide refactor exception**: when a mechanical change (rename, shared type change) has a wide blast radius and cannot keep a single slice green, order it as **expand (old and new coexist) -> migrate (in batches by package / directory) -> contract (delete the old)**
- ticket descriptions **avoid concrete file paths and code snippets** (they go stale); exception: decision fragments produced by a prototype that are more precise than prose (state machine / reducer / schema / type shape) may be inlined with a note that they come from the prototype
- **hand the slice list to the user for a pass before release** (granularity, blocking edges, whether to merge or split), iterate until approved

## Status column values

`not started` / `in progress` / `merged` / `blocked`. Use words, not emoji.

**Do not confuse this with the status board**: here it is **ticket-level** status (one value per ticket within a requirement). The `dev status` column of the status board `docs/status.md` is **requirement-level** (`planning` / `in progress` / `submitted` / `pending verification`, see [`../pm/group-conventions.md`](../pm/group-conventions.md) "Status board"), the two granularities differ and are not required to use the same words.

## de dispatch prompt template

`de` is a subagent generated dynamically when DM dispatches, it does not get a separate skill file: role boundary, workflow and reporting format are all embedded into the prompt from this template. The name is fixed as `de`, one message launches all calls of the same batch; the worktree and branch are **pre-created by DM** before dispatching. Dispatch is always **background/asynchronous** (mechanism mapping in [`../pm/group-conventions.md`](../pm/group-conventions.md) "Organization"), DM ends the round right after sending (return control), acceptance is woken by the completion notification, do not wait in place for results.

**Dispatch parameters (written on the call, not into the prompt text)**: `context: "fresh"` (does not inherit the parent conversation) + `timeoutMs` (team discipline #6: fill in when writing the ticket, device / long chains >= 45 min and report in segments, pure document tickets may be short). **Mutually exclusive numbering (team discipline #14)**: before starting / dispatching **actually check the three directory kinds** (`docs/pd/spec/` / `docs/dev/plan/` / `docs/test/report/`) to confirm the number is free + **announce to PM/TM as soon as the number is taken** + conflicts are arbitrated by "**first to write to disk wins**", the other side **bumps the number and leaves a trace**. **Device exclusivity (team discipline #13)**: device tickets must write four things into the prompt: **pre-probe / `trap` reset / install registration / reinstall or hand back**; device windows are scheduled by DM (the same device must not run two tickets concurrently). The template body also lists `timeoutMs` as one line, so DM does not miss it when copying the ticket.

**Retired knob prohibition**: for tickets that touch a **retired knob / switch**, **never design a "same build, flip the knob" route** (flipping config leaves behaviour unchanged while the self-proof key may still report 1 => **false negative**, more dangerous than a "stuck at 0" key); **A/B must use two builds** (register each artifact SHA and leave a trace in the source line), and **never use a self-proof key as evidence that the arm was switched**.

**Red-arm criterion prohibition**: **red-arm (reverse-arm) criteria must distinguish "expected red" from "environment / compile failure"**: an `expect FAIL` criterion **must not test only `rc!=0`** (a broken build also yields rc!=0 => **teeth that bite nothing**); **compile / environment failure is always HARD FAIL (never record OK)**, so write the criterion as "**build succeeds AND rc==1**" or an equivalent explicit form, plus a reverse arm "make it fail to compile on purpose => must be ERROR". **Same family**: group discipline `G16` (criteria / tools decoupled from their own conditions). **Self-check per dispatch**: does the red arm separate "expected red / environment failure"? Does the criterion declare `worktree@commit`? Is the self-proof key compared on both sides of the **capability surface**?

**Companion action (G16, reading vs fact)**: whenever a reading is used as a conclusion, give evidence for **both sides, "reading surface vs fact surface"** (without the fact side you may not conclude): treating an arrival rate as a processed frame rate, a self-proof key reporting 1 while behaviour is unchanged, or an install failing yet reporting success are all cases where the reading side holds and the fact side does not.

**Companion action (G16, artifact timing)**: artifact-type gates (reading an APK / `.so` / build artifact) **must build first, then run the gate** (the reverse order reads a stale artifact, so a red gate is a timing error, not a defect) => **fixed acceptance order: build -> gate -> (real-device items)**. **Running a "scope gate" while new files are untracked gives a false green** (`git diff` does not list untracked files) => **re-run after committing**.

```
Role: de, a development implementation agent, not DM; you have no subagent tool and must not dispatch subagents, when dispatch is needed only write "suggest DM dispatch X". Report only to DM, do not contact other departments or other de.

Boundary: work only inside the worktree below, only change files within this ticket's scope; commit only, no push (merging is done by DM); do not change the plan, do not do independent code review (that belongs to the test department), no external communication, no polling; evidence goes into the report message, do not write evidence files to disk.

Context:
- Timeout: timeoutMs = {milliseconds} (must already be filled at dispatch, do not leave empty, do not rely on the default)
- worktree: {worktree absolute path} (DM has created it, cd in directly, do not run git worktree add again; from then on all edits and commands use this absolute path, writing any file or committing anything in the main repo worktree is strictly forbidden)
- Branch: feat/{ticket-id} (already exists)
- Build/test command: {command}
- Locating material: {original error text + file:line + reproduction command or log path; three items mandatory for fix / diagnostic tickets, team discipline #12, if missing DM gathers evidence before dispatching}
- Static assertions: {command or assertion list} (run for every ticket, do not save for the end)
- Four steps before commit: git log -1 to check the base -> if HEAD has advanced, git diff <base>..HEAD -- this ticket's files (non-empty means stop and report) -> explicit-path git add (no -A) -> never amend/reset someone else's commit
- Unique numbering: {ticket-id} is assigned by DM, if you find a conflict with the plan or another running ticket, stop and ask
- Device (filled only for device tickets, team disciplines #8/#13): pre-probe (`adb devices -l` / `settings get system user_rotation` / `settings get system accelerometer_rotation` / `lastUpdateTime` of `dumpsys package <pkg>`) + the script uses **`trap` to reset unconditionally** (rotation / SP / switch state) + at the end **reinstall the canonical build or hand back explicitly** + **install registration (package name/SHA/time)**; **the concurrency form only writes "this ticket is running"**, device exclusivity windows are scheduled by DM (the same device must not run two tickets concurrently)

Ticket: {ticket-id}
Goal: {end-to-end behavior}
Full ticket text: {paste the full text or the plan file path}
Acceptance criteria: {item by item}
TDD seam: {seam}

Workflow:
1. Read the ticket above, enter the worktree
2. Write code per implement (https://raw.githubusercontent.com/mattpocock/skills/main/skills/engineering/implement/SKILL.md), run the tdd red-green loop at the predetermined seam (https://raw.githubusercontent.com/mattpocock/skills/main/skills/engineering/tdd/SKILL.md)
3. Use code-review (https://raw.githubusercontent.com/mattpocock/skills/main/skills/engineering/code-review/SKILL.md) for the Standards axis self-review (does it match repo conventions)
4. Run the static assertions given above (do not save for the end)
5. Commit inside this worktree's branch, reply to DM directly in the format below, do not save a separate document file

Reporting format:
ticket / branch + worktree / commit sha / changed files (file:line) / TDD record (Red, Green, Refactor, Build, Assert, one line each) / static assertion output / self-review record (Standards axis conclusion) / acceptance criteria (item by item, pass or blocked with reason) / **device state reset self-proof** (device tickets only: paste the post-reset readback: `user_rotation` / `accelerometer_rotation` / SP / `lastUpdateTime` of `dumpsys package`, and state clearly **which package was installed, when, and when it was reset**) / blocked items, out-of-bounds requests, leftovers.

Criterion: build passes, unit tests pass, static assertions pass, self-review Standards axis passes, commit sha is verifiable.
```
