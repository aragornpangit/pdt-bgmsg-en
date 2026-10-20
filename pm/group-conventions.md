# PDT group conventions: communication / documents / naming / status board

This file is the **single source of truth** for the PDT group.

## Organization structure

```
pdt-group (top-level department, owner pm: user entry point + product manager)
├── pd   product department   manager pm  → SPEC (writes it themselves)
│   └── pde × n               derived on demand (requirement talks / evidence / verification), reports only to pm, pm consolidates results
├── dev  development dept     manager dm  → development plan and progress (writes it themselves)
│   └── de × n                single ticket implementation → .worktrees/{ticket}
├── test testing department   manager tm  → test report (writes it themselves)
│   └── te × n                test case execution
└── scoutM Reconnaissance dept manager scoutM → research report (docs/scout/, writes it themselves)
    └── se × n                   scout-engineers (several derived on demand)
```

- Each manager **writes their own department's documents**; pm may derive `pde` to do work on demand (product requirement conversations / evidence gathering / verification / material organization), but the delivered document is still consolidated by pm and written by pm
- **pm is both group owner and product manager**: takes on user requests, sets up and manages dm / tm sessions, arbitrates at the top level
- `de` / `te` / `se` / `pde` are all derived by **the manager who spawned them** as n parallel subagents according to actual need (n = number of parallel tickets / parallel batches / research sub-questions, may be 1, do not force-split just to reach a count); **a subagent reports only to the manager that spawned it and accepts assignments only from that manager**; pm consolidates the work results of the subagents it spawned
- **Subagents dispatched in the background**: **every role dispatches subagents in background/async mode** (including pm's pde and scoutM's se), **return control immediately after dispatching**, keep the session able to receive herdr messages, no waiting in place, polling, or blocking waits for subagent returns; subagent completion triggers acceptance or review via a completion notification (task-notification / native wakeup). Multiple batches may be held at once. Map the specific switch to the agent tool you use (e.g. background run parameter, async subagent, native parallel subagents)
- **Maximize efficiency**: parallelize whenever possible; the only full text is in "Team disciplines" item 11 (named carriers: codebuddy's team mode, claude-code's dynamic workflow); troubleshooting is parallel too: gather evidence for several hypotheses at once, do not guess serially on one line
- Direct channel between managers (pm / dm / tm negotiate horizontally); when the two sides cannot agree, pm arbitrates
- **Research goes to scoutM**: when pm / dm face technology selection, competitor benchmarking, or pre-refactor assessment, assign the work to the scoutM session (first locate the pane ID where `scoutM` lives per the "Communication" section), **do not spin up se yourself**; scoutM's conclusion flows back into the SPEC or development plan in one sentence. If the scoutM session does not exist, pm first starts one per [`team-bootstrap.md`](team-bootstrap.md)

## Communication: direct herdr, this is the main channel

**Between pm / dm / tm, and from pm / dm → scoutM, always use targeted `herdr` direct connection** (CLI syntax in the herdr skill itself), talk directly, no third-party relay. scoutM only accepts dispatches from the user or pm / dm, and does not chime in on its own.

- Status, progress, conclusions, defect send-backs, arbitration requests all go through herdr messages
- **The inbox file is self-contained**: one-sentence summary + artifact path + requested action (the herdr message is only the doorbell)
- **Locate the partner before sending a message (hard requirement)**: first run `herdr tab list` to list the labels of all current tabs, confirm the recipient label is in the list; **a label is only for finding the person, it cannot be used directly as the message target** (reason and the full three steps below).

- **herdr agent targets only accept an "agent name" or a "pane ID", not a tab label** (measured: `herdr agent get scoutM` returns `agent_not_found`, while the `scoutM` label is clearly present in `herdr tab list`). Full three steps to locate a partner:

  1. `herdr tab list` → find the `tab_id` for the recipient label
  2. `herdr pane list` → find the `pane_id` under that `tab_id`
  3. **Delivery: prefer `herdr agent prompt`** (the target may be an agent name or a pane_id, provided that pane is recognised as an agent, i.e. its `agent` field in `herdr agent list` is not empty):

     ```bash
     herdr agent prompt <agent-name-or-pane_id> "<one-line summary + artifact path + requested action>"
     ```

     **Why this one**: it goes through the agent's structured channel (it honours bracketed paste and appends one encoded Enter), so it **does not steal focus, does not bring the window to the foreground, and does not depend on whether enter gets swallowed**. **The cost**: it still writes into that pane's stdin, because herdr has no out-of-band message bus (`herdr api` only offers `snapshot` / `schema`), so "never touching the input box" is not achievable; what is achievable is "no focus change, atomic delivery, multi-line safe". codebuddy has been in the agent kind list since herdr 0.9.3 and can use this channel (the older "you must use the raw pane channel" conclusion is void).

     **Pre-send check**: the pane's `agent_status` must be `idle` / `done`; when it is `blocked` (stuck on a permission dialog), handle it manually first, since `agent prompt` returns `agent_blocked` instead of answering the dialog by mistake.

     **Fallback (panes not recognised as an agent)**: `herdr pane send-text <pane_id> "<message>"` + `herdr pane send-keys <pane_id> enter`; multi-line messages must use `herdr-ml`, otherwise an embedded newline is treated as Enter and submits early. Re-check `agent_status` two seconds later; if it is still not `working`, send one extra enter, and never resend the body (to prevent duplicate delivery).

     **Token-saving delivery format (hard)**:

     - **Body goes into a file, the message carries three things only**: one-line summary + artifact path + requested action. Never paste long bodies, logs or diffs into the message
     - **Do not add `--wait`**: it waits synchronously and pulls the recipient's whole turn of output back, which burns tokens for nothing
     - **Do not verify delivery with `agent read`**: reading pane output floods the context; ask the recipient for a one-line receipt instead ("got it, will report in X minutes")
     - **Merge a batch into one message**: say several things in one delivery instead of splitting them into a back-and-forth

**All message bodies go through an inbox file (hard)**: the herdr message is only the doorbell saying "you have mail"; never stuff the body into the message.

1. **Write the file**: drop it into the recipient's inbox at `docs/<recipient-role>/inbox/{seq}-{direction}-{semantic-slug}-{YYMMDD-HHMM}-v{N}.md`. Direction = sender2recipient, one of `pm2dm` / `dm2pm` / `pm2tm` / `tm2pm` / `dm2tm` / `tm2dm` / `pm2scoutM` / `scoutM2pm` / `scoutM2dm` / `dm2scoutM`. `{seq}` **reuses the requirement chain's number, never a new one** (see team discipline item 14); for convention talk unrelated to a requirement, use the number of the requirement in flight
2. **Ring the doorbell**: `herdr agent prompt <target> "you have a new message {filename} under docs/<recipient-role>/inbox/"` (fallback as above: the pane channel)
3. **Recipient**: read the file, reply with a one-line receipt, then **delete the file immediately** (read and delete; the inbox may only ever hold unread messages)
4. **Not committed**: the inbox is a runtime channel, not a deliverable. The project's `.gitignore` must contain `docs/*/inbox/`; `mkdir -p` the inbox when it is missing

  For sessions started with `herdr agent start <name>`, that `<name>` can be used directly; agents started manually by the user are unnamed, and only `pane_id` works. **Never write a label or ID from memory, never hardcode an ID** (IDs may change each session, always look them up at use time).
- **Recipient label not in `herdr tab list` ⇒ do not send**: first complete the session per "Sessions and labels" (completion flow in [`team-bootstrap.md`](team-bootstrap.md)), confirm readiness, then send; do not retry repeatedly against a session that does not exist
- No polling: upstream updates are triggered by the user / pm, or checked once at the start of this round's task
- **Do not write status report documents**: progress is synced via messages, not by writing files for mutual awareness

## Reporting conventions (what should reach the user)

**By default, do not report**: criterion details, ticket and document registration, scheduling and execution details, technical solution choices, tool/script issues, minor convention adjustments; partners close the loop among themselves.

**Only two kinds of things are reported to the user**:

1. **Needs the user to decide**: product conventions (requirements, defaults, display), priority trade-offs, destructive or irreversible operations, cost versus experience trade-offs
2. **Cross-department matters that need PM arbitration**: DM / TM disagreements, cross-ticket conflicts, convention conflicts

**Mechanism (must happen first)**:

- Partners' messages **go to PM by default**, not directly to the user
- When one of the two categories above is hit, **PM escalates to the user**; when escalating, **state in one sentence** what needs deciding, which options exist, and what the cost is, without the intermediate process
- PM **no longer relays** partner receipts one by one; progress is summarized by milestone or on demand, in one go

## Document whitelist (hard requirement)

**Only the following kinds of document files are allowed.** Anything that needs recording is merged into an existing document, **no new document types** (no session logs, no summaries, no "report" documents, no README for code).

| Document | Path | Limit | Owner |
|---|---|---|---|
| SPEC | `docs/pd/spec/{seq}-spec-{semantic-slug}-{timestamp}-v{N}.md` | 80 lines (baseline) | PM |
| Development plan and progress | `docs/dev/plan/{seq}-devPlan-{semantic-slug}-{timestamp}-v{N}.md` | 50 lines (baseline) | DM |
| Test report | `docs/test/report/{seq}-testReport-{semantic-slug}-{timestamp}-v{N}.md` | 40 lines (baseline) | TM |
| Evidence directory | `docs/test/evidence/{seq}-testReport-{semantic-slug}-ev/` | as needed | TM |
| Research report | `docs/scout/{seq}-scout-{semantic-slug}-{timestamp}-v{N}.md` | as needed | **ScoutM** (PM dispatches, does not write it themselves) |
| Inbox message | `docs/<recipient-role>/inbox/{seq}-{direction}-{semantic-slug}-{timestamp}-v{N}.md` | as needed (deleted once read) | sender writes, recipient deletes after reading |
| Handoff document | `docs/pd/handoff/pm-{semantic-slug}-{timestamp}-v{N}.md`<br>`docs/dev/handoff/dm-{semantic-slug}-{timestamp}-v{N}.md`<br>`docs/test/handoff/tm-{semantic-slug}-{timestamp}-v{N}.md` | 25 lines (baseline) | PM / DM / TM each write their own |
| Domain terminology | `CONTEXT.md` | as needed | PM |
| Status board (fallback, see below) | `docs/status.md` | one line per requirement | three parties each own their column |

- Deprecated: `docs/adr/` (architecture decisions merged into the "Design decisions" section of the development plan), `docs/dev/report/`, `docs/test/plan/`, `docs/dev/evidence/`, `docs/handoff/`. Three were later restored: `docs/scout/` restored for the research report (**only ScoutM's research artifacts go in**; other roles' research conclusions still go directly into the SPEC); `docs/test/evidence/` restored for "host-side evidence committed to the repo" (**for TM archiving only**); handoff restored with per-department directories, see the "Handoff" section below (**the original repo-root `docs/handoff/` is still unused**)
- **Two layers of evidence** (per the TM evidence-in-repo ruling): ① **conclusion evidence inline**: give `file:line` or grep output in the report, no separate evidence file on disk; ② **host-side machine-check evidence committed**: `selftest` logs, machine-check output etc. go to `docs/test/evidence/` (reports are updated in place within a version and their filenames carry a timestamp and `v{N}`; using that directly as the directory name would mean the directory changes whenever the report version is bumped, breaking the evidence chain; so the directory name **drops the timestamp and version number** and appends an `-ev` suffix), the directory looks like `{seq}-testReport-{semantic-slug}-ev/`, with topics as subdirectories under it; **evidence belongs to the requirement chain, one directory per seq, the directory does not change when the report version is bumped**, and the report inlines "command + summary + path"; summaries may be truncated, but must keep **key lines sufficient for recomputation** such as assertion counts, failing lines, `rc`, and command lines. `de` / `te` do not write to disk; evidence is archived by TM / DM
- **Scripts / probes are not governed by the whitelist**: probes / unit tests / static assertion scripts are **tools required for recomputation** (item 7) ⇒ **they go into the repo code tree** `tools/<ticket>/` (not into build artifacts); "evidence inline" **only covers artifacts** (screenshots / sidecars / logs), **not tools**: "scripts not committed ⇒ rebuild on demand" = not recomputable (rebuilding depends on the original script ⇒ circular)
- **Delete sections with no content outright**, do not write "none" / "N/A" / "to be added" placeholders

## Writing disciplines

0. **Do not write documents every round** (hard requirement): only touch documents when there is substantive output or a convention change (new SPEC, version bump, plan change, report finalization); daily rounds do not write to disk, progress and conclusions go via herdr messages. Documents are artifacts to be read, not a work log
1. **One table row per item**, do not explain in prose what the table already says; conclusion in one sentence, no preamble, no background introduction, no methodology
2. **Finish in one pass**: no rework or polishing except for factual errors
3. **The line limit is a baseline, not a target** (deliverable line counts may be relaxed somewhat): when over, delete redundancy first; if nothing can be deleted and it still must be exceeded, register the reason in one line at the top of the file, do not hack away information
4. **Time budget**: cumulative document writing for the same feature must not exceed 10% of implementation time. If you find yourself repeatedly revising documents, stop and go do the work

## Expression (all roles)

**Governs three places: speech (messages), writing files (reports / plans / notes), and code comments.** Target reader = someone who knows a little software engineering but has not worked on this project (including the user). Role SKILL.md files no longer restate this section, they keep a pointer only.

- **Speak plain language only, conclusion first, then evidence, highlight the key points**: no jargon pile-up, the first sentence gives the conclusion (pass / fail / needs a decision), the basis follows
- **No invented terms**: no self-made abbreviations or internal jargon; when a technical term must be used, **add a plain-language explanation the first time it appears**
- **One idea per paragraph**, short sentences; give a concrete example rather than an abstract conclusion
- **Explain three things clearly**: what this is, why it matters, what happens if you do it; write it as an action someone can follow (who does it, what to do, where, and what counts as done right)
- **Numbers carry their source**: where the number comes from, what it measures, when it was measured
- **Comments also speak human**: write for "the next person to read this code", state clearly "what this section does and why it is done this way", no references only this team understands
- **Work tickets**: called ticket in English, "work order" in speech, not abbreviated to "ticket stub"

**Supporting action (must be done before sending)**: ask yourself "**can someone who understands programming but has not worked on this project understand it?**" If not, revise, then send. Reports aimed at the user additionally run the jargon grep self-check: **the only detailed version (jargon mapping table + self-check command + two exceptions) is in [`communication-style.md`](communication-style.md)**; roles no longer each maintain a copy of the table.

## Team disciplines (cross-role, hard requirement)

> **The single carrier for technical disciplines = this file** (the former `docs/test/DISCIPLINES.md` / `docs/group/decisions.md` are deprecated). Each role's `SKILL.md` keeps one line of pointer plus the role-specific execution surface, and **no longer restates the full text** (the "two-sided loop" duplication is cancelled, to prevent two copies from drifting).
> **Meta-discipline**: a discipline must **come with a supporting action** (turn "remember to comply" into "an action that must happen first"); writing only "do not do X" counts as **incomplete and can be rejected**. Each item must also have a **reason** (fixes get copied; the reason is what explains "why this cannot be skipped").
> **Admission bar (two questions, failing either ⇒ the discipline does not stand)**: ① **check the source**: which **real failure mode** does it prevent (being able to state the failure mechanism is enough, **no need to record project names or ticket numbers**; concrete instances are carried by each role's `disciplines.md`; if you cannot even name the mode ⇒ a discipline that has never been violated, which by the dual of "untested is not evidence" does not count); ② **check the reason**: if at writing time you are stuck on "what is the reason" ⇒ there probably is no real source, rethink what it is guarding against.
> **Dual**: **a mechanism that has never been run, and a discipline that has never been violated, both do not count** ⇒ a mechanism must be marked with "first actual-run evidence"; a discipline must be marked with "the failure mode it prevents".
> **⚠ Numbering system (registered 2026-10-12; settled when TM pointed out "one number carrying two different disciplines")**: the numbering in this document's discipline table is the "**group discipline**" numbering, and is **two independent systems** from the "**global discipline**" numbering in `~/.pi/agent/AGENTS.md` (example: **global discipline 15 = atomic commit discipline**, while **group discipline 15 = four checks before cleanup**) ⇒ **references must carry their carrier**: in this document write **`G<n>`** (G = Group, e.g. `G15`), global disciplines are written `A<n>` (A = AGENTS, e.g. `A15`). Historical references to the old items (1–12) in this document **are not rewritten** (item 11: no global replacement of history); all new references carry the carrier prefix. ⚠ **The overlap range is all of 1–15** (global table 1–20, group table currently 1–15 ⇒ any bare number ≤ 15 **hits both**) ⇒ do not assume "only 13–15 conflict"; old text is kept, but **any reference that drives an action** (opening a ticket / arbitration / acceptance) must carry the carrier when newly written.

| # | Discipline | Supporting action (must happen first) | Same family |
|---|---|---|---|
| 1 | No other action may be inserted between `git add` and `git commit`; `commit -m … -- <path>` bypasses the index | **New files (untracked) must be `git add`ed first** (otherwise `pathspec … did not match`, `rc=1`, **nothing is committed**); review with `git diff --cached --name-only` before committing | item 5 |
| 2 | Self-checks and reported numbers must **actually run the shortest sufficient substring**, never extrapolate from memory | **Re-run the same command** after a fix; **any reported number must be preceded by actual run output** | items 4 / 5 / 9 |
| 3 | A criterion's "truth value" must not be taken from **the output of the object under test** (prevent circular reasoning); nor may it **share a blind spot** with the object under test. **Before using probe output as a criterion, first check the probe's position in the function and its trigger condition**: if the probe sits at the end of the function, "zero output" may mean either "not called" or "called and then stuck midway", and **when the criterion cannot distinguish these two explanations, do not use it alone to draw a conclusion** | Truth must be **independently recomputable**; others' pre-labeling serves only as a **lower-bound estimate** and must not be the final judgment of recall. Before setting a criterion, `grep -n` the probe string's definition site and write the position into the criterion; when the two explanations must be distinguished, add a separate independent criterion (e.g. also look at the **per-field triple** rather than the total, otherwise a total of 0 is fooled by "the object was never built at all") | item 4 |
| 4 | **Three-piece reporting**: whenever a number is reported externally, give "① the measurement convention + ② the registration point + ③ the original source (file / round)" at the same time. Reason = the most common form of "padding for green" is not fabrication, but **swapping in a better-looking number** (the convention can be correct, the motive cannot be judged) | Write the three pieces out before writing the document; missing any one counts as an **incomplete report that can be rejected** (rejection only says "item ② is missing, please add", no accusation) | items 2 / 9 |
| 5 | **Announcing completion requires verification numbers**: any claim that "X is in place" must be accompanied by `grep -c` (or equivalent) output (**0 first → write → 1 after**), plus the "source of the pattern string"; **the verification target must equal the delivery target**. Reason = commit verifies "is it on disk", grep verifies "is it written into place"; missing either gate means silent failure | The writing script **echoes the key substring**, and verification runs `grep -c` only against that echoed string, never retyped by hand; `git add` first → then verify → both `git diff --cached` and `git diff` empty → then commit; claiming "repo-wide / remote state" requires `fetch` first then verify, or explicitly limiting the claim to the local copy | items 1 / 2 |
| 6 | **Dispatch timeout is mandatory**: dispatching a subagent (new / resume) must explicitly set `timeoutMs`, **filled in when the ticket is written**, never relying on the default. Reason = the default 30 min silently kills a long-running task the moment it hits, wasting all prior work with nobody knowing | Make `timeoutMs` a fixed ticket field; device / long-chain work ≥ 45 min with segmented reporting; pure document tickets may be short | meta-discipline |
| 7 | **Cite means fetch the source**: numbers, version numbers, licenses etc. must come from the primary artifact, "grep first, then write"; a claim without a source does not count; **when a ticket cites a SPEC / criterion item, verify the original text word by word before writing**, never from memory or compressed context. **Dispatch communication has three more hard requirements**: ① **verify and paste output before citing a path or line number**: run `ls` / `grep -n` once, paste the output into the ticket, do not write "remember to verify"; ② **pass command fragments and special characters through a file**, using `herdr pane send-text "$P" "$(cat <file>)"`, **never inline them into a shell double-quoted string** (backticks / `$` / angle brackets get eaten by command substitution, and **after inlining drops characters `send` still returns rc=0**); ③ **confirm delivery by "the other side restating the key constraint"**: the `send` return code, `agent_status` changes, and locally grepping the pane content **all three can be false negatives** (long messages go into the steering queue / message list, not the visible area; on this basis "not delivered" was misjudged 5 times, while all had actually been received); with no receipt, **ask**, do not guess and do not resend on that basis | ① before dispatching, run an original-text grep for each cited item and paste the matching sentence into the ticket, give primary sources for performance numbers; ② write the message body to a file before `send-text`, **commands in the body always go in an indented block or reference a file path, never inline**; ③ put key rulings **at the end** of the body, and the receiver's receipt must restate that paragraph; **cannot restate ⇒ judged truncated, resend directly** (this is the only reliable delivery criterion) | item 4 |
| 8 | **Review conclusions must declare the reviewed version**: when diffing / contract-checking / numerically checking others' artifacts, the conclusion must carry the reviewed object's `commit` or the three elements; comparing across points in time without checking the version mistakes "after correction" for "before correction" | Before concluding, take the object's SHA (`git rev-parse`) and write it into the conclusion; the dividing line between "trace left" and "residue" = whether the old literal is explicitly marked "void / superseded"; **machine-check / assertion commands must declare the execution tree** (worktree path + commit SHA); the same command at the repo root and in a ticket worktree can give **opposite** results (measured 2026-10-12: `grep -c 'screenOrientation="portrait"'` flips from 0 to 1 with cwd) | item 4 |
| 9 | **Keyword hits must state the scope**: reporting "hit count / 0 hits" must include the scope | The reporting format is fixed as the "**hit count + scope + command**" triple; **a hit count without a scope = invalid evidence and can be rejected**; when the scope spans tickets ⇒ **bucket by ticket** | items 2 / 4 |
| 10 | **Commit identity = role**: the author/committer name must be `userid(role)`, role ∈ {pm, dm, tm, scoutM, user}; for document commits the first line of the message also marks "writing role + ticket number"; **correction / retraction instructions must carry a verifiable anchor (file:line or ticket number); if the anchor is missing, obtain it first and then execute; a paraphrased summary is only a lead**. Reason = the author of `53413ba` could not be determined (a single identity repo-wide, only content can be reverse-engineered afterwards); a four-hop paraphrased correction instruction nearly made ScoutM change a document over a nonexistent accusation | Machine check = the repo pre-commit identity gate (`core.hooksPath` **must be an absolute path**: a relative path does not resolve in a worktree ⇒ the gate fails entirely (there is a fix instance); self-check = `git config --show-origin --get core.hooksPath` must be an absolute path and the directory must exist); `commit-msg` checks the same to cover merge (merge does not trigger pre-commit); **rejected by the gate (rc=1) ⇒ change identity and resubmit; `git commit --no-verify` bypass is forbidden** (bypassing disables the gate entirely and silently; PM arbitrates); **a server-side gate is not feasible** (server hooks on a self-hosted GitLab origin must be configured by an admin in the UI; GitHub does not support git server hooks) ⇒ rely on "reduce false hits + post-hoc audit"; the audit script `tools/audit-identity.sh` (scans `git log --format='%an'` for violation counts, non-blocking); on receiving a correction instruction, grep for the anchor first; if it cannot be found, ask the dispatcher | items 7 / 8 |
| 11 | **Maximize parallelism**: work that can be parallelized must be opened as the maximum number of subagents or carried by a workflow (e.g. codebuddy team, dynamic workflow); only real resource constraints (devices, writes to the same worktree, sequential state) allow reducing the parallelism count. Failure mode = user feedback "work is too slow", root cause is a conservative parallelism habit (start one and see, do not open more without enough justification), not insufficient resources | Before dispatching, ask "can this batch take one more ticket"; when a ticket can be split into mutually independent subtasks, split it and send them in the same message; dispatch is always background/async (see "Organization structure"), wrap up the round right after dispatching | meta-discipline |
| 12 | **Locating material travels with the ticket**: fix / diagnosis tickets must include three things: **the original error text + file:line (or grep hit output) + the reproduction command or log path**; defect send-backs use the same format. Failure mode = user feedback "locating is too slow", root cause is tickets missing material causing multiple back-and-forth Q&A rounds, with locating time spent asking for material instead of finding the cause | Before dispatching, self-check whether the three are complete; if missing, gather evidence first then dispatch (aligned with item 7 "verify and paste output before citing a path"); on receiving an under-specified ticket, the receiver asks for material before starting, no guessing out of thin air | item 7 |
| 13 | **Real-device occupation must be reset and self-proven**: whenever a real device is occupied (installing a package / collecting data / verifying), at the end you must **reset the device state** (rotation lock / SP / switch states / installed package) and **prove it yourself**; do not hand the next user or the user a device that has been modified. Failure mode = **the user directly sees an abnormal UI** (measured 2026-10-12: after installing a verification package for 041, `user_rotation=1` + `accelerometer_rotation=0` were not reset, and the user actually saw "the UI turned landscape"; in the same round there was also "a verification package installed but not registered, the user thought the official build was running") | **Fixed dispatch template fields**: device pre-check (rotation / SP / package SHA readback) + unconditional `trap` reset + at the end **reinstall the canonical package or explicitly hand back**; **one fixed acceptance item**: readback self-proof after reset (`user_rotation` / `accelerometer_rotation` / SP / `lastUpdateTime` from `dumpsys package`) + **debug artifacts on device external storage** (`Android/media/<pkg>/files/`: probe trigger files and `bcr_probe/` etc.; **residue must be 0** (found the 2nd time the same day 2026-10-12, the first being `bcr_probe/` slowing down the user's preview)); the receipt must state "which package was installed, when, and when it was reset" | item 7 |
| 14 | **seq number allocation is mutually exclusive (number anchor)**: `{seq}` is a **cross-department shared anchor** (shared by the SPEC / development plan / test report chain); before taking a number you must actually check and avoid in-flight ticket numbers. Failure mode = **same number, different meaning** (measured 2026-10-12: PM and DM concurrently took `049` ⇒ the SPEC and dev ticket collided, corrected afterwards by write order; also in the same round DM's `git mv` left an index residue, and the TM-side evidence that `-- <explicit path>` did not sweep it away conversely shows item 15 is effective) | Before taking a number, run one actual check across all three directories (`ls docs/pd/spec/ docs/dev/plan/ docs/test/report/ | grep -E '<candidate number>'`); **announce the number immediately** to the parties involved; conflicts are arbitrated by "**first to write wins**", the holder keeps it, the other side bumps its number and leaves a trace (no global replacement, item 11) | item 11 |
| 15 | **Four checks before cleaning up a worktree / shared resource**: whenever cleaning up a worktree, branch, or shared resource that another role may still be using, four checks must come first. Failure mode = **deleting an in-flight workspace by mistake** (measured 2026-10-12: when DM cleaned `.worktrees/049-01`, because of a "ticket number ↔ worktree mismatch" it luckily deleted the one that was **already merged** and avoided the in-flight one; DM admitted it had **not checked "is this branch already merged"** before (luck, not process, not reproducible)) | Before cleanup, run each check and paste it into the cleanup register: ① **check for live de** (list async runs / running tickets); ② **verify ownership by the "ticket number → worktree mapping"** (not just by directory name; if a mismatch exists, check by ticket number); ③ **confirm no other role is using it** (evidence / artifacts); ④ **confirm the branch is merged** (`git branch --merged`, **never clean up an unmerged one**) | item 13 |
| 16 | **Family item: criteria / tools decoupled from their own conditions** (covering five coupling surfaces: **object / environment / execution tree / batch / capability surface**): criteria, assertions, probes, and self-proof keys **must be bound to their own conditions**: truth must not be taken from the output under test (object) / machine checks must declare `worktree@commit` (execution tree) / same-batch collection must come from the same source (batch) / self-proof keys must be compared on both sides with the capability surface (capability surface). Failure mode = **false red / false green / false teeth / non-zero false values**. **6 measured cases on 2026-10-12**: ① compile failure `rc=90` recorded as OK as an "expected red" (false teeth) ② `grep -c` flips with cwd (0/1) ③ a retired knob's self-proof key still reports 1 (non-zero false value) ④ `install` fails yet reports success / SHA mismatch yet still installed ⑤ a constant-0 key (`preview_render_frames`) read as "measured 0" ⑥ header-comment convention drift (wrote vertical 1080×1920, measured 4096×2304), found across departments (test side 3 / dev side 2 / de self-check 1) ⇒ **not sporadic**, hence named as a family | Three fixed requirements for dispatch templates: ① **the red-arm criterion must distinguish "expected red / environment failure"** (environment failure ⇒ HARD FAIL, **must not be recorded as OK**); ② **criteria must declare `worktree@commit` (execution tree)**; ③ **self-proof keys must be compared "on both sides" with the capability surface** (do not only judge "has a value"; do not use a self-proof key as evidence that "the arm has been switched") | same family as G3 / G8 (G3 = truth must not be taken from the object under test; G8 = reviews must declare the version) |

**Gate effectiveness boundary (PM-measured register, three anchors, SHA authoritative)**:

| Range | Status | Anchor |
|---|---|---|
| ≤ `c9a21add` | gate running (14 consecutive `userid(dm)`) | `c9a21add`…`b2bf13ab` (2026-10-03 12:16–12:33) |
| after the rename → before `f996b85d` | **gate not running at all**: `core.hooksPath` points at the pre-rename dead directory `…/bcr-nihui/.git/hooks` (that directory no longer exists); all 137 unlabeled commits on 2026-10-04 are consistent with this | `f996b85d` (2026-10-05 03:18) fixed to the absolute path `.githooks`, and the 3 commits after it are all compliant |
| ≥ `f996b85d` | gate effective | none |

**Historical identities are not retroactive**: of 1484 commits repo-wide, 1441 have no role label. **Do not rewrite history**: labels can only be guessed from content and time, which turns "known to be unfindable" into "pretending it was found", the opposite of discipline 3; and it would require force-pushing all 4 remotes and breaking every other clone, with zero benefit. Historical segments needing accountability go through an **offline attribution register** (whoever claims, registers), **without touching git objects**.

**Register correction**: `docs/dev/plan/009.md` once recorded "48 of the last 50 commits are `panxiaoyong`"; the actual measurement is **44** (convention = author name of the last 50 commits; source `git log -50 --format='%an' | sort | uniq -c`, actual run 2026-10-05 03:33 → 44 panxiaoyong / 3 tm / 2 scoutM / 1 dm). The difference is because those 50 are a **rolling window**; the ratio changed after DM and TM landed compliant commits, it is not a statistical error.

**de identity auto-injection: promised but never installed (measured)**: discipline 10 originally said "de is auto-injected `userid(dm)` by the global includeIf (`.git/worktrees/**`), no manual addition needed". Actual measurement: `~/.gitconfig` **has only the two sections `[user]` and `[credential]`, with no includeIf at all**, while the global `user.name = panxiaoyong` ⇒ de committing in a worktree **is necessarily rejected by the gate**, which is exactly the real cause that forced `--no-verify`. DM has been dispatched to install it. **Before installation**: de must manually pass `-c user.name='userid(dm)'`; **after installation**: this item reverts to "auto-injected, no manual addition needed". Supporting action = after installation, re-verify end to end inside a worktree (legal identity passes the gate with rc=0, illegal identity is rejected with rc=1), not just checking whether the config was written.

## Fail cleanly (general design preference)

**For any tool / script / write operation, fail cleanly: either everything succeeds or nothing changes** (no half-finished products, no "half-written" intermediate state).

- **How**: writing documents / modifying files always "assert first, then write"; when the assertion fails the file was never changed, no cleanup needed, and nobody can misread it
- **Same family**: do not retry repeatedly when a tool cannot connect; `git commit -- <path>` failing loudly on untracked files is better than silently missing files
- **Criterion**: after a failure the workspace / files are exactly as before the operation (machine-checkable: failure ⇒ `git status` identical to before)

## File naming (unified format for the five artifact types)

`{seq}-{prefix}-{semantic-slug}-{YYMMDD-HHMM}-v{N}.md`

- **`{seq}` goes at the start of the filename, following the SPEC's sequence number**: PM takes the number when creating the SPEC (starting at 001, mutual exclusion per team discipline 14); **the development plan and test report reuse the seq of the same SPEC**, no separate number; research reports are not attached to the SPEC chain and use their own cumulative serial number
- **Five prefix choices**: `spec` (SPEC) / `devPlan` (development plan and tickets) / `testReport` (test report) / `scoutM` (research report, written by ScoutM, dispatched by PM) / **direction** (inbox messages, values in the "Communication" section, e.g. `pm2dm`). **Handoff documents are not one of these five**, see the "Handoff" section for naming
- **Semantic slug**: English words joined by hyphens, no more than 5 words (e.g. `user-sync`), expressing the target semantics of this file
- **Timestamp**: the moment this version was first written to disk, format `YYMMDD-HHMM` (two-digit year, two-digit month, two-digit day, two-digit hour, two-digit minute)
- **Version number `v{N}`, starting from v1**: **small in-version updates** (appending cases, per-ticket status, progress) modify the same file in place, no rename; **substantive revisions bump the version into a new file** (new timestamp, `v{N+1}`, same seq), and the old file's header is marked in place "superseded by vX" (trace versus residue: the old literal is explicitly voided, do not use deletion to pretend it never happened)
- **Referencing same-chain artifacts**: locate by `{seq}-{prefix}-*.md` (timestamp and version number are not part of the reference; glob fills them in)
- `mkdir -p` when the directory is missing

## Status board (fallback only when herdr is unavailable)

By default **do not create or maintain** `docs/status.md`. Only when herdr is completely unavailable and mutual awareness must go through files, create this table (header only):

```markdown
# Requirement status board

| SPEC | Title | Development status | Test status | Updated |
|---|---|---|---|---|
| 001-user-sync | User sync | Implementing | To be written | 2026-08-10 |
```

- One line per requirement, the updated field uses an absolute date
- **PM maintains two columns**: `SPEC` (seq) and `Title`; **DM maintains `Development status`** (`Planning` / `Implementing` / `Submitted` / `To verify`); **TM maintains `Test status`** (`To be written` / `To run` / `Pass` / `Blocked` / `Needs fix`)
- **PM is read-only on the other two columns** (does not change status for DM / TM), but **cross-department arbitration conclusions are written into this table by PM**
- From the seq you can derive the three same-chain artifact types: `docs/pd/spec/{seq}-spec-*.md`, `docs/dev/plan/{seq}-devPlan-*.md`, `docs/test/report/{seq}-testReport-*.md`
- When the status board disagrees with reality, reality wins and the board is updated to match

## Handoff

**Daily cross-role transfer goes via herdr messages; no handoff documents.** Handoff documents are produced only **when using mattpocock's [`/handoff`](https://raw.githubusercontent.com/mattpocock/skills/main/skills/productivity/handoff/SKILL.md) for session self-handoff**, and in that case **must be written to disk**, not left only in the session:

| Role | Path | Filename | Owner |
|---|---|---|---|
| pm | `docs/pd/handoff/` | `pm-{semantic-slug}-{YYMMDD-HHMM}-v{N}.md` | PM writes their own |
| dm | `docs/dev/handoff/` | `dm-{semantic-slug}-{YYMMDD-HHMM}-v{N}.md` | DM writes their own |
| tm | `docs/test/handoff/` | `tm-{semantic-slug}-{YYMMDD-HHMM}-v{N}.md` | TM writes their own |

- **One per role, write only your own**: do not write for other roles, and do not have three roles co-write one document
- **Semantic slug**: English words joined by hyphens, no more than 5 words, summarizing the topic of this handoff (e.g. `user-sync`)
- **Timestamp**: the moment of writing to disk, `YYMMDD-HHMM` (two-digit year, month, day, hour, minute)
- **Version number `v{N}` starting from v1**: small in-session additions modify the same file in place, no rename; when the handoff content is substantially replaced, create a new version file (new timestamp, `v{N+1}`), and mark the old file's header **in place "superseded by vX"**
- **No `{seq}`**: handoff documents are not one of the five artifact types (SPEC / development plan / test report / research report / inbox message) and are not attached to the requirement chain seq
- **Content**: 25 lines baseline, delete redundancy first when over; write only the minimum sufficient information for the successor; commit first, then push
- **Subagents (`de` / `te`) do not write handoffs**; the manager consolidates everything
- `mkdir -p` when the directory is missing

## Sessions and labels

Session names are fixed as `pm` / `dm` / `tm`, the label set is `{pm, dm, tm, scoutM}` (pm is the group entry point, dm / tm are started by pm; scoutM is started by pm on demand, or the user issues research directly in its session). **When sending messages, these are the four labels to look for in `herdr tab list`**: managers look up `pm` / `dm` / `tm`; dispatching research looks up `scoutM`; only when found do you continue to resolve the pane ID (steps in the "Communication" section); if not found, complete the session first.

- **`sessions` in `.pdt/team.json` registers the four labels**: `dm` / `tm` are written by pm at creation; `scoutM` is started by pm on demand and registered the same way after starting (fields isomorphic to `dm` / `tm`)

- **Runtime base and communication priority**: prefer **herdr as the terminal runtime base**, with each role's session running inside a herdr session where possible (one session per role, labels above); between sessions, **prefer targeted herdr messaging** (delivery method in the "Communication" section: prefer `herdr agent prompt`, and only fall back to the pane channel for panes not recognised as an agent), not file polling, not cross-session relay. Only fall back when herdr is unavailable (fallback in the manual mode of [`team-bootstrap.md`](team-bootstrap.md)).

## Cross-skill reference disciplines

- **Pointer only**: communication rules, document whitelist, naming rules, and herdr CLI syntax exist in one place only (this file); everything else points to it
- **Self-contained**: specific items that get rewritten repeatedly, such as verification methods and assertion semantics, are written in full by each skill, noted as "same name and meaning as on the X side", not pointing at each other, to avoid silent drift when the other side changes its table
