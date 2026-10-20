# pdt-group skill inventory

The mattpocock engineering skills used by the group's manager roles (pm / dm / tm), assigned to the pd / dev / test departments according to **mattpocock's semantics**. `de` / `te` are subagents dynamically generated at the manager's dispatch stage and do not get their own skill files (templates in `dm/templates.md`, `tm/templates.md`). Each role's SKILL.md writes only the few it wires up, and does not restate them here.

**Upstream repository**: [`mattpocock/skills`](https://github.com/mattpocock/skills) (GitHub, `main` branch). Every skill name in the table below links to its raw `SKILL.md` address on `main` (raw root `https://raw.githubusercontent.com/mattpocock/skills/main/`); click through to read the upstream original. **Upstream is the single source of truth**.

**Type axis** (mattpocock's original text): **user-invoked** can only be triggered by a human typing a command, and its job is **orchestration**; **model-invoked** can be typed by a human or reached automatically by an agent, and carries **reusable discipline**. user-invoked may call model-invoked, and **must never call another user-invoked**.

## pd product department (PM): requirement -> SPEC

| Skill | Type | Semantics | Who runs it |
|---|---|---|---|
| [`grilling`](https://raw.githubusercontent.com/mattpocock/skills/main/skills/productivity/grilling/SKILL.md) | MODEL | The interrogation primitive, the shared base for `grill-me` / `grill-with-docs` / `triage` / `wayfinder` / `improve-codebase-architecture` | PM |
| [`grill-me`](https://raw.githubusercontent.com/mattpocock/skills/main/skills/productivity/grill-me/SKILL.md) | USER | Interrogation for non-code scenarios | user types it |
| [`grill-with-docs`](https://raw.githubusercontent.com/mattpocock/skills/main/skills/engineering/grill-with-docs/SKILL.md) | USER | Interrogation + synchronously update `CONTEXT.md` | user types it, PM takes over |
| [`domain-modeling`](https://raw.githubusercontent.com/mattpocock/skills/main/skills/engineering/domain-modeling/SKILL.md) | MODEL | Proactively build and sharpen the domain model: challenge terminology, stress-test boundary cases, update `CONTEXT.md` | PM |
| [`prototype`](https://raw.githubusercontent.com/mattpocock/skills/main/skills/engineering/prototype/SKILL.md) | MODEL | A throwaway prototype answers a design question (state/logic kind, or a UI variant) | PM (decides whether to do it) |
| [`to-spec`](https://raw.githubusercontent.com/mattpocock/skills/main/skills/engineering/to-spec/SKILL.md) | USER | Conversation -> SPEC, published to the tracker (**already internalized as [`spec-format.md`](spec-format.md)**) | PM |
| [`writing-for-agents`](https://raw.githubusercontent.com/mattpocock/skills/main/skills/productivity/writing-for-agents/SKILL.md) | MODEL | Write documents "for agents to read" (a SPEC is exactly this kind) | PM |
| [`to-questionnaire`](https://raw.githubusercontent.com/mattpocock/skills/main/skills/productivity/to-questionnaire/SKILL.md) | USER | Decisions you cannot answer yourself -> a questionnaire for the person who can answer them | user types it, PM takes over |
| [`triage`](https://raw.githubusercontent.com/mattpocock/skills/main/skills/engineering/triage/SKILL.md) | USER | An issue goes through the triage role state machine (needs label configuration) | user types it, PM takes over |
| [`wait-what`](https://raw.githubusercontent.com/mattpocock/skills/main/skills/productivity/wait-what/SKILL.md) | USER | When a message does not land, have the agent restate it in `CONTEXT.md` vocabulary | user types it |
| [`handoff`](https://raw.githubusercontent.com/mattpocock/skills/main/skills/productivity/handoff/SKILL.md) | USER | Session self-handoff (when `/handoff` is used, write to this department's `handoff/` directory) | PM |

## dev development department (DM): design -> implementation -> merge

| Skill | Type | Semantics | Who runs it |
|---|---|---|---|
| [`codebase-design`](https://raw.githubusercontent.com/mattpocock/skills/main/skills/engineering/codebase-design/SKILL.md) | MODEL | Shared vocabulary and discipline for deep modules (lots of behavior, small interface, placed on a clean seam) | DM |
| [`wayfinder`](https://raw.githubusercontent.com/mattpocock/skills/main/skills/engineering/wayfinder/SKILL.md) | USER | Split work larger than a single session into a map of decision tickets, resolve them one by one | DM |
| [`to-tickets`](https://raw.githubusercontent.com/mattpocock/skills/main/skills/engineering/to-tickets/SKILL.md) | USER | plan/spec -> tracer-bullet tickets + blocking edges (**already internalized as "slicing rules" in [`../dm/templates.md`](../dm/templates.md)**) | DM |
| [`implement`](https://raw.githubusercontent.com/mattpocock/skills/main/skills/engineering/implement/SKILL.md) | USER | Implement per spec/tickets, drive `tdd`, wrap up with `code-review` | `de` (dynamically derived by DM, see dm/templates.md) |
| [`tdd`](https://raw.githubusercontent.com/mattpocock/skills/main/skills/engineering/tdd/SKILL.md) | MODEL | Red-green loop, one vertical slice at a time | `de` (dynamically derived by DM) |
| [`code-review`](https://raw.githubusercontent.com/mattpocock/skills/main/skills/engineering/code-review/SKILL.md) | MODEL | **Standards axis** self-review (repo conventions + Fowler bad-smell baseline) | `de` (dynamically derived by DM) |
| [`improve-codebase-architecture`](https://raw.githubusercontent.com/mattpocock/skills/main/skills/engineering/improve-codebase-architecture/SKILL.md) | USER | Scan for deepening opportunities -> HTML report -> interrogate the one selected (**condition**: PM explicitly asks for tech debt cleanup) | DM |
| [`wizard`](https://raw.githubusercontent.com/mattpocock/skills/main/skills/engineering/wizard/SKILL.md) | MODEL | Generate an interactive bash wizard that walks a human through steps only a human can do (credentials / CI / migration) (**condition**) | DM |
| [`setup-pre-commit`](https://raw.githubusercontent.com/mattpocock/skills/main/skills/misc/setup-pre-commit/SKILL.md) | MODEL | Husky + lint-staged + type checking + tests (**condition**) | DM |
| [`git-guardrails-claude-code`](https://raw.githubusercontent.com/mattpocock/skills/main/skills/misc/git-guardrails-claude-code/SKILL.md) | MODEL | Block dangerous git commands (**condition**) | DM |
| [`migrate-to-shoehorn`](https://raw.githubusercontent.com/mattpocock/skills/main/skills/misc/migrate-to-shoehorn/SKILL.md) | MODEL | Migrate `as` assertions in test files (**condition**: TS projects) | DM |
| [`handoff`](https://raw.githubusercontent.com/mattpocock/skills/main/skills/productivity/handoff/SKILL.md) | USER | Session self-handoff (when `/handoff` is used, write to this department's `handoff/` directory) | DM |

## test department (TM): independent review -> defect localization

| Skill | Type | Semantics | Who runs it |
|---|---|---|---|
| [`code-review`](https://raw.githubusercontent.com/mattpocock/skills/main/skills/engineering/code-review/SKILL.md) | MODEL | **Spec + Standards dual-axis** independent review (parallel subagents, no cross-contamination), corresponds to `XX-CODE` test cases | `te` (dynamically derived by TM, see tm/templates.md) |
| [`diagnosing-bugs`](https://raw.githubusercontent.com/mattpocock/skills/main/skills/engineering/diagnosing-bugs/SKILL.md) | MODEL | Diagnosis loop for hard defects / performance regressions (red -> minimize -> hypothesis -> instrumentation -> fix -> regression) | `te` (dynamically derived by TM) |
| [`handoff`](https://raw.githubusercontent.com/mattpocock/skills/main/skills/productivity/handoff/SKILL.md) | USER | Session self-handoff (when `/handoff` is used, write to this department's `handoff/` directory) | TM |

> `de` and `te` share `code-review` but on different axes: `de` does a Standards self-review, `te` does a Spec + Standards independent review, each serving as independent evidence for the other. Both are subagents dynamically generated at the manager's dispatch stage, and their skill wiring is written into the dispatch prompt templates (`dm/templates.md`, `tm/templates.md`).

## Cross-role and not included

| Skill | Type | Owner | Note |
|---|---|---|---|
| [`ask-matt`](https://raw.githubusercontent.com/mattpocock/skills/main/skills/engineering/ask-matt/SKILL.md) | USER | pm | Router: ask "which skill / process should I use" |
| [`research`](https://raw.githubusercontent.com/mattpocock/skills/main/skills/engineering/research/SKILL.md) | MODEL | scoutM | Not wired since 2026-09-23; wired on 2026-09-29 to scoutM, the manager of the Reconnaissance department; since 2026-10-20 it reaches the execution surface: **ScoutM writes the research skill address into the prompt when dispatching `se`**, and `se` researches per that skill plus anti-hallucination cross-verification, output goes to `docs/scout/` |
| [`setup-matt-pocock-skills`](https://raw.githubusercontent.com/mattpocock/skills/main/skills/engineering/setup-matt-pocock-skills/SKILL.md) | USER | one-time | Configure issue tracker / triage labels / doc directory, run once per repo |

**Not included** (unrelated to the PDT process, or personal toolchains): `teach`, `scaffold-exercises`, `pr` (write the PR body), `implement-spec`, `retro` (upstream has finalized it and moved it into the `engineering/` bucket, not needed by the process for now), plus the not-yet-finalized `claude-handoff`, `loop-me`, `setup-ts-deep-modules`, `writing-beats`, `writing-fragments`, `writing-shape` in the `in-progress/` bucket.

> Upstream bucket structure (actual state on `main`): `productivity` / `engineering` / `misc` / `in-progress`. The links in this table all point to `main`; if upstream renames or moves buckets the links will 404, and it is enough to fix this one table to the new paths.

## Maintenance

When skills are added or removed, change only this table; the pointer sections in each role's SKILL.md stay consistent with it.

> Trace of removed references: `resolving-merge-conflicts` was once the basis for dm's conflict resolution, and **upstream v1.3.0 has archived it out of the plugin** (explicitly saying there is no replacement, and the agent handles conflicts directly); the skill file is no longer on `main`, only an archive note remains. So this repo removes that reference, and the dm side switches to a practice write-up with no citation (trace each hunk by intent, never `--abort`).
