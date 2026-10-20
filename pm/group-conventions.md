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
└── scout-m Reconnaissance dept manager scout-m → research report (docs/scout/, writes it themselves)
    └── se × n                   scout-engineers (several derived on demand)
```

- Each manager **writes their own department's documents**; pm may derive `pde` to do work on demand (product requirement conversations / evidence gathering / verification / material organization), but the delivered document is still consolidated by pm and written by pm
- **pm is both group owner and product manager**: takes on user requests, sets up and manages dm / tm sessions, arbitrates at the top level
- `de` / `te` / `se` / `pde` are all derived by **the manager who spawned them** as n parallel subagents according to actual need (n = number of parallel tickets / parallel batches / research sub-questions, may be 1, do not force-split just to reach a count); **a subagent reports only to the manager that spawned it and accepts assignments only from that manager**; pm consolidates the work results of the subagents it spawned
- **Subagents dispatched in the background**: **every role dispatches subagents in background/async mode** (including pm's pde and scout-m's se), **return control immediately after dispatching**, keep the session able to receive herdr messages, no waiting in place, polling, or blocking waits for subagent returns; subagent completion triggers acceptance or review via a completion notification (task-notification / native wakeup). Multiple batches may be held at once. Map the specific switch to the agent tool you use (e.g. background run parameter, async subagent, native parallel subagents)
- **Maximize efficiency**: parallelize whenever possible; the only full text is in "Team disciplines" item 11 (named carriers: codebuddy's team mode, claude-code's dynamic workflow); troubleshooting is parallel too: gather evidence for several hypotheses at once, do not guess serially on one line
- Direct channel between managers (pm / dm / tm negotiate horizontally); when the two sides cannot agree, pm arbitrates
- **Research goes to scout-m**: when pm / dm face technology selection, competitor benchmarking, or pre-refactor assessment, assign the work to the scout-m session (first locate the pane ID where `scout-m` lives per the "Communication" section), **do not spin up se yourself**; scout-m's conclusion flows back into the SPEC or development plan in one sentence. If the scout-m session does not exist, pm first starts one per [`team-bootstrap.md`](team-bootstrap.md)

## Communication: direct herdr, this is the main channel

**Between pm / dm / tm, and from pm / dm → scout-m, always use targeted `herdr` direct connection** (CLI syntax in the herdr skill itself), talk directly, no third-party relay. scout-m only accepts dispatches from the user or pm / dm, and does not chime in on its own.

- Status, progress, conclusions, defect send-backs, arbitration requests all go through herdr messages
- **The inbox file is self-contained**: one-sentence summary + artifact path + requested action (the herdr message is only the doorbell)
- **Locate the partner before sending a message (hard requirement)**: first run `herdr tab list` to list the labels of all current tabs, confirm the recipient label is in the list; **a label is only for finding the person, it cannot be used directly as the message target** (reason and the full three steps below).

- **herdr agent targets only accept an "agent name" or a "pane ID", not a tab label** (measured: `herdr agent get scout-m` returns `agent_not_found`, while the `scout-m` label is clearly present in `herdr tab list`). Full three steps to locate a partner:

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

1. **Write the file**: drop it into the recipient's inbox at `docs/<recipient-role>/inbox/{spec-seq}-{direction}-{semantic-slug}-{YYMMDD-HHMM}-v{N}.md`. Direction = sender2recipient, one of `pm2dm` / `dm2pm` / `pm2tm` / `tm2pm` / `dm2tm` / `tm2dm` / `pm2scout-m` / `scout-m2pm` / `scout-m2dm` / `dm2scout-m`. `{spec-seq}` is per G14 (**never a new number**); for convention talk unrelated to a requirement, use the number of the requirement in flight
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
| SPEC | `docs/pd/spec/{spec-seq}-spec-{semantic-slug}-{timestamp}-v{N}.md` | 80 lines (baseline) | PM |
| Development plan and progress | `docs/dev/plan/{spec-seq}-devPlan-{semantic-slug}-{timestamp}-v{N}.md` | 50 lines (baseline) | DM |
| Test report | `docs/test/report/{spec-seq}-testReport-{semantic-slug}-{timestamp}-v{N}.md` | 40 lines (baseline) | TM |
| Evidence directory | `docs/test/evidence/{spec-seq}-testReport-{semantic-slug}-ev/` | as needed | TM |
| Research report | `docs/scout/{seq}-scout-{semantic-slug}-{timestamp}-v{N}.md` | as needed | **scout-m** (PM dispatches, does not write it themselves) |
| Inbox message | `docs/<recipient-role>/inbox/{spec-seq}-{direction}-{semantic-slug}-{timestamp}-v{N}.md` | as needed (deleted once read) | sender writes, recipient deletes after reading |
| Handoff document | `docs/pd/handoff/pm-{semantic-slug}-{timestamp}-v{N}.md`<br>`docs/dev/handoff/dm-{semantic-slug}-{timestamp}-v{N}.md`<br>`docs/test/handoff/tm-{semantic-slug}-{timestamp}-v{N}.md` | 25 lines (baseline) | PM / DM / TM each write their own |
| Domain terminology | `CONTEXT.md` | as needed | PM |
| Status board (fallback, see below) | `docs/status.md` | one line per requirement | three parties each own their column |

- Deprecated, do not recreate: `docs/adr/` (architecture decisions merged into the "Design decisions" section of the development plan), `docs/dev/report/`, `docs/test/plan/`, `docs/dev/evidence/`, and `docs/handoff/` at the repo root (handoffs moved into per-department directories, see "Handoff")
- **Two layers of evidence** (per the TM evidence-in-repo ruling): ① **conclusion evidence is inlined**: a `file:line` or grep result in the report is enough, no separate evidence file; ② **host-side machine-check evidence goes into the repo**: `selftest` logs and machine-check output land in `docs/test/evidence/{spec-seq}-testReport-{semantic-slug}-ev/`, with the topic as a subdirectory; the directory name **drops the timestamp and version** (otherwise a report version bump changes the directory and breaks the evidence chain), **one seq, one directory**. The report inlines "command + summary + path"; the summary may be truncated but must keep assertion counts, failing lines, `rc` and the command line, that is, **the key lines needed to recompute**. `de` / `te` do not write to disk; TM / DM archive the evidence
- **Scripts / probes are not governed by the whitelist**: probes / unit tests / static assertion scripts are **tools required for recomputation** => **they go into the code tree** at `tools/<ticket>/` (not into build output). "Inline evidence" governs artifacts only (screenshots / sidecars / logs), **never tools**: a script that is not in the repo can only be rebuilt on demand, and rebuilding depends on the original script, which is circular
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
> **Admission bar (two questions, failing either ⇒ the discipline does not stand)**: ① **check the source**: which **real failure mode** does it prevent (being able to state the failure mechanism is enough, **no need to record project names or ticket numbers**; concrete instances are carried by each role's own reference document (e.g. `dm/disciplines.md`); if you cannot even name the mode ⇒ a discipline that has never been violated, which by the dual of "untested is not evidence" does not count); ② **check the reason**: if at writing time you are stuck on "what is the reason" ⇒ there probably is no real source, rethink what it is guarding against.
> **Dual**: **a mechanism that has never been run, and a discipline that has never been violated, both do not count** ⇒ a mechanism must be marked with "first actual-run evidence"; a discipline must be marked with "the failure mode it prevents".
> **Numbering system**: disciplines in this document are numbered **`G<n>`** (G = Group); disciplines carried by other documents (e.g. a global `AGENTS.md`) are written `A<n>`. Any bare number <= 15 matches both systems, so **new references always carry the carrier prefix**; old text is not rewritten and never globally replaced.

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
| 10 | **Commit identity = role**: the author/committer name must be `userid(role)`, role ∈ {pm, dm, tm, scout-m, user}; document commits put "writing role + ticket id" on the first line of the message; **correction / retraction instructions must carry a verifiable anchor (file:line or ticket id); fill a missing anchor before acting, and treat a relayed summary as a clue only**. Reason = with a single repo-wide identity the author can only be inferred from content afterwards, so events cannot be judged; a many-times-relayed correction once nearly made the recipient edit a document over an accusation that did not exist | Machine check = the repo pre-commit identity gate (`core.hooksPath` **must be absolute**: a relative path does not resolve inside a worktree, so the gate silently stops working; self-check `git config --show-origin --get core.hooksPath` must be absolute with the directory present); `commit-msg` covers merges (a merge does not trigger pre-commit); **rejected by the gate (rc=1) => fix the identity and re-commit, `--no-verify` is forbidden** (bypassing disables the gate silently); **a server-side gate is not achievable** (self-hosted GitLab needs an admin, GitHub has no git server hooks) => rely on "reduce false hits + post-hoc audit" (the audit script is built in the target repo, scanning `git log --format='%an'` for violations, non-blocking); on a correction instruction, grep the anchor first and ask the dispatcher when it is not found | item 7 / 8 |
| 11 | **Maximize parallelism**: work that can be parallelized must be opened as the maximum number of subagents or carried by a workflow (e.g. codebuddy team, dynamic workflow); only real resource constraints (devices, writes to the same worktree, sequential state) allow reducing the parallelism count. Failure mode = user feedback "work is too slow", root cause is a conservative parallelism habit (start one and see, do not open more without enough justification), not insufficient resources | Before dispatching, ask "can this batch take one more ticket"; when a ticket can be split into mutually independent subtasks, split it and send them in the same message; dispatch is always background/async (see "Organization structure"), wrap up the round right after dispatching | meta-discipline |
| 12 | **Locating material travels with the ticket**: fix / diagnosis tickets must include three things: **the original error text + file:line (or grep hit output) + the reproduction command or log path**; defect send-backs use the same format. Failure mode = user feedback "locating is too slow", root cause is tickets missing material causing multiple back-and-forth Q&A rounds, with locating time spent asking for material instead of finding the cause | Before dispatching, self-check whether the three are complete; if missing, gather evidence first then dispatch (aligned with item 7 "verify and paste output before citing a path"); on receiving an under-specified ticket, the receiver asks for material before starting, no guessing out of thin air | item 7 |
| 13 | **Real-device occupation must be reset and self-proven**: whenever a real device is occupied (install / capture / verification), **reset the device state** at the end (rotation lock / SP / switch states / installed package) and **self-prove it**; never hand the next user a device someone else changed. Failure mode = **the user directly sees an abnormal screen** (rotation lock left off after a verification install => the UI turns landscape), plus "a verification build was installed but never registered, so the user thought the official build was running" | **Fixed fields in the dispatch template**: pre-flight probe (rotation / SP / package SHA read-back) + `trap` unconditional reset + **restore the canonical build or hand over explicitly** at the end; **one fixed acceptance item**: read back and self-prove after the reset (`user_rotation` / `accelerometer_rotation` / SP / `dumpsys package` `lastUpdateTime`) + **debug artifacts left on device external storage must be 0** (probe trigger files and temp directories under `Android/media/<pkg>/files/`); the receipt states "which package, installed when, reset when" | item 7 |
| 14 | **`{spec-seq}` unique source = the SPEC's id (numbering anchor)**: `{spec-seq}` is a **cross-department shared anchor**; **only PM takes a number, when creating the SPEC**; DM's ticket document, TM's test report and inbox messages all reuse it and **never take their own**. Failure mode = **same number, different meaning** (PM and DM concurrently took the same number => the SPEC and the dev ticket collided; the root cause is that two roles could both take numbers, so this item is now **single-point numbering**) | PM **announces the seq to DM and TM via herdr immediately** after creating the SPEC; before writing to disk, DM/TM check that the seq in the filename matches the SPEC they work from (`SEQ=<the SPEC id>; ls docs/pd/spec/ | grep "$SEQ"`); on a mismatch **stop and report**, never bump the number yourself, never do a global replacement (see "Numbering system") | item 4 |
| 15 | **Four checks before cleaning up a worktree / shared resource**: whenever cleaning up a worktree, a branch, or a shared resource another role may still be using, run four checks first. Failure mode = **deleting a worktree that was still in flight** (during cleanup, because "ticket id vs worktree" were misaligned, the one deleted happened to be the **already-merged** one while the in-flight one was missed; the operator admitted never checking "is this branch merged", so it was luck rather than process and cannot be reproduced) | Before cleaning, run each check and paste it into the cleanup record: ① **check live de** (list async runs / in-flight tickets); ② **map ticket id -> worktree to confirm ownership** (not just by directory name; when misalignment exists, go by ticket id); ③ **confirm no other role is using it** (evidence / artifacts); ④ **confirm the branch is merged** (`git branch --merged`; **never clean up an unmerged branch**) | item 13 |
| 16 | **Family item: criteria / tools decoupled from their own conditions** (covering five coupling surfaces: **object / environment / execution tree / batch / capability surface**): criteria, assertions, probes and self-proof keys **must be bound to their own conditions**: the truth value must not come from the output under test (object) / machine checks must declare `worktree@commit` (execution tree) / same-batch captures must share a source (batch) / a self-proof key must be compared on both sides of the capability surface (capability surface). Failure mode = **false red / false green / teeth that bite nothing / non-zero false values**: a compile failure recorded as "expected red" and marked OK, `grep -c` flipping with cwd, a retired knob's self-proof key still reporting 1, `install` failing yet reporting success, a stuck-at-0 key read as "measured 0", a header convention that disagrees with the measured size => multiple cases across departments, so it is named as a family | Three fixed requirements for the dispatch template: ① **the red arm must separate "expected red / environment failure"** (environment failure => HARD FAIL, **never record OK**); ② **criteria must declare `worktree@commit` (execution tree)**; ③ **self-proof keys must be compared on both sides of the capability surface** (never just "has a value"; never use a self-proof key as evidence the arm was switched) | same family as G3 / G8 (G3 = truth must not come from the object under test; G8 = review must declare the version) |

**Commit identity gate and historical commits**: the gate once failed entirely because `core.hooksPath` was a relative path (a relative path does not resolve inside a worktree, it must be absolute) and became effective again once it was made absolute; most historical commits carry no role label, and they are **never rewritten and never force-pushed**; accountability for those segments goes through an offline attribution register (whoever claims, registers; git objects are not touched). `de` is **not** currently auto-injected with `userid(dm)`, so a de commit must pass `-c user.name='userid(dm)'` by hand; once the includeIf is installed, re-verify end to end inside a worktree (legal identity passes with rc=0, illegal identity is rejected with rc=1) rather than just checking that the config was written.

## Fail cleanly (general design preference)

**For any tool / script / write operation, fail cleanly: either everything succeeds or nothing changes** (no half-finished products, no "half-written" intermediate state).

- **How**: writing documents / modifying files always "assert first, then write"; when the assertion fails the file was never changed, no cleanup needed, and nobody can misread it
- **Same family**: do not retry repeatedly when a tool cannot connect; `git commit -- <path>` failing loudly on untracked files is better than silently missing files
- **Criterion**: after a failure the workspace / files are exactly as before the operation (machine-checkable: failure ⇒ `git status` identical to before)

## File naming (unified format for the five artifact types)

`{spec-seq}-{prefix}-{semantic-slug}-{YYMMDD-HHMM}-v{N}.md`

(The one exception: research reports are not attached to the SPEC chain and still use `{seq}`, see the next item)

- **`{spec-seq}` has exactly one source, the SPEC's id**: see G14 (only PM takes the number, when creating the SPEC; DM / TM / inbox all reuse it). A SPEC version bump (`v{N+1}`) **keeps `{spec-seq}` unchanged**; research reports are not attached to the SPEC chain and still use `{seq}` (a cumulative serial number of scout-m's own, starting at 001)
- **Five prefix choices**: `spec` (SPEC) / `devPlan` (development plan and tickets) / `testReport` (test report) / `scout` (research report, written by scout-m, dispatched by PM) / **direction** (inbox messages, values in the "Communication" section, e.g. `pm2dm`). **Handoff documents are not one of these five**, see the "Handoff" section for naming
- **Semantic slug**: English words joined by hyphens, no more than 5 words (e.g. `user-sync`), expressing the target semantics of this file
- **Timestamp**: the moment this version was first written to disk, format `YYMMDD-HHMM` (two-digit year, two-digit month, two-digit day, two-digit hour, two-digit minute)
- **Version number `v{N}`, starting from v1**: **small in-version updates** (appending cases, per-ticket status, progress) modify the same file in place, no rename; **substantive revisions bump the version into a new file** (new timestamp, `v{N+1}`, same seq), and the old file's header is marked in place "superseded by vX" (trace versus residue: the old literal is explicitly voided, do not use deletion to pretend it never happened)
- **Referencing same-chain artifacts**: locate by `{spec-seq}-{prefix}-*.md` (timestamp and version number are not part of the reference; glob fills them in)
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
- From `{spec-seq}` you can derive the three same-chain artifact types: `docs/pd/spec/{spec-seq}-spec-*.md`, `docs/dev/plan/{spec-seq}-devPlan-*.md`, `docs/test/report/{spec-seq}-testReport-*.md`
- When the status board disagrees with reality, reality wins and the board is updated to match

## Handoff

**Daily cross-role transfer goes via herdr messages; no handoff documents.** Handoff documents are produced only **when using mattpocock's [`/handoff`](https://raw.githubusercontent.com/mattpocock/skills/main/skills/productivity/handoff/SKILL.md) for session self-handoff**, and in that case **must be written to disk**, not left only in the session:

| Role | Path | Filename | Owner |
|---|---|---|---|
| pm | `docs/pd/handoff/` | `pm-{semantic-slug}-{YYMMDD-HHMM}-v{N}.md` | PM writes their own |
| dm | `docs/dev/handoff/` | `dm-{semantic-slug}-{YYMMDD-HHMM}-v{N}.md` | DM writes their own |
| tm | `docs/test/handoff/` | `tm-{semantic-slug}-{YYMMDD-HHMM}-v{N}.md` | TM writes their own |

- **One per role, write only your own**: do not write for other roles, and do not have three roles co-write one document
- **Semantic slug / timestamp / version**: rules as in "File naming" (slug <= 5 words, `YYMMDD-HHMM`, `v{N}` from v1, small in-place updates, a substantive rewrite bumps the version)
- **No `{spec-seq}`**: handoff documents are not one of the five artifact types and are not attached to the requirement chain seq
- **Content**: 25 lines baseline, delete redundancy first when over; write only the minimum sufficient information for the successor; commit first, then push
- **Subagents (`de` / `te`) do not write handoffs**; the manager consolidates everything
- `mkdir -p` when the directory is missing

## Sessions and labels

Session names are fixed as `pm` / `dm` / `tm`, the label set is `{pm, dm, tm, scout-m}` (pm is the group entry point, dm / tm are started by pm; scout-m is started by pm on demand, or the user issues research directly in its session). **When sending messages, these are the four labels to look for in `herdr tab list`**: managers look up `pm` / `dm` / `tm`; dispatching research looks up `scout-m`; only when found do you continue to resolve the pane ID (steps in the "Communication" section); if not found, complete the session first.

- **`sessions` in `.pdt/team.json` registers the four labels**: `dm` / `tm` are written by pm at creation; `scout-m` is started by pm on demand and registered the same way after starting (fields isomorphic to `dm` / `tm`)

- **Runtime base and communication priority**: prefer **herdr as the terminal runtime base**, with each role's session running inside a herdr session where possible (one session per role, labels above); between sessions, **prefer targeted herdr messaging** (delivery method in the "Communication" section: prefer `herdr agent prompt`, and only fall back to the pane channel for panes not recognised as an agent), not file polling, not cross-session relay. Only fall back when herdr is unavailable (fallback in the manual mode of [`team-bootstrap.md`](team-bootstrap.md)).

## Cross-skill reference disciplines

- **Pointer only**: communication rules, document whitelist, naming rules, and herdr CLI syntax exist in one place only (this file); everything else points to it
- **Self-contained**: specific items that get rewritten repeatedly, such as verification methods and assertion semantics, are written in full by each skill, noted as "same name and meaning as on the X side", not pointing at each other, to avoid silent drift when the other side changes its table
