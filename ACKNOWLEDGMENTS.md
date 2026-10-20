# Acknowledgments

## Matt Pocock and `mattpocock/skills`

The methodological skeleton of this project is built directly on **Matt Pocock**'s open-source skill set. His work is the precondition for this project existing, and we have learned a great deal from it.

- Upstream repo: [`mattpocock/skills`](https://github.com/mattpocock/skills) (MIT License)
- Author profile: <https://github.com/mattpocock>
- Repo description: Skills for Real Engineers. Straight from my .agents directory.

### Which skills are actually referenced

The table below is **explicit reference**, not a blanket tribute. Every row has a corresponding wiring point in this project, and each skill name links straight to the source on the upstream `main` branch:

| Location in this project | Upstream skill referenced |
|---|---|
| `pm`'s interview step | [`grilling`](https://raw.githubusercontent.com/mattpocock/skills/main/skills/productivity/grilling/SKILL.md) |
| `scoutM`'s research method (anti-hallucination cross-validation, internalized as step 3 of the scoutM SKILL.md workflow) | [`research`](https://raw.githubusercontent.com/mattpocock/skills/main/skills/engineering/research/SKILL.md) |
| `pm`'s SPEC format | [`to-spec`](https://raw.githubusercontent.com/mattpocock/skills/main/skills/engineering/to-spec/SKILL.md), internalized as `pm/spec-format.md` |
| `dm`'s design vocabulary | [`codebase-design`](https://raw.githubusercontent.com/mattpocock/skills/main/skills/engineering/codebase-design/SKILL.md) |
| `dm`'s slicing convention | [`to-tickets`](https://raw.githubusercontent.com/mattpocock/skills/main/skills/engineering/to-tickets/SKILL.md), internalized as the "slicing details" in `dm/templates.md` |
| `de`'s implementation loop | [`implement`](https://raw.githubusercontent.com/mattpocock/skills/main/skills/engineering/implement/SKILL.md), [`tdd`](https://raw.githubusercontent.com/mattpocock/skills/main/skills/engineering/tdd/SKILL.md), [`code-review`](https://raw.githubusercontent.com/mattpocock/skills/main/skills/engineering/code-review/SKILL.md) (Standards axis self-review) |
| `te`'s independent review | [`code-review`](https://raw.githubusercontent.com/mattpocock/skills/main/skills/engineering/code-review/SKILL.md) (Spec + Standards dual axis), [`diagnosing-bugs`](https://raw.githubusercontent.com/mattpocock/skills/main/skills/engineering/diagnosing-bugs/SKILL.md) |
| Session self-handoff of the three manager roles (when `/handoff` is used, files land in that department's `handoff/` directory) | [`handoff`](https://raw.githubusercontent.com/mattpocock/skills/main/skills/productivity/handoff/SKILL.md) |
| General primitives (each role draws on them as needed; **ownership and invocation type are in [`pm/inventory.md`](pm/inventory.md)**) | [`writing-for-agents`](https://raw.githubusercontent.com/mattpocock/skills/main/skills/productivity/writing-for-agents/SKILL.md), [`domain-modeling`](https://raw.githubusercontent.com/mattpocock/skills/main/skills/engineering/domain-modeling/SKILL.md), [`prototype`](https://raw.githubusercontent.com/mattpocock/skills/main/skills/engineering/prototype/SKILL.md), [`wayfinder`](https://raw.githubusercontent.com/mattpocock/skills/main/skills/engineering/wayfinder/SKILL.md), [`wizard`](https://raw.githubusercontent.com/mattpocock/skills/main/skills/engineering/wizard/SKILL.md), [`triage`](https://raw.githubusercontent.com/mattpocock/skills/main/skills/engineering/triage/SKILL.md), [`ask-matt`](https://raw.githubusercontent.com/mattpocock/skills/main/skills/engineering/ask-matt/SKILL.md) |

The full ledger (including invocation type and executor) is in [`pm/inventory.md`](pm/inventory.md).

### What we gained

Ordered by impact, largest first; all of these are identifiable concrete changes, not pleasantries:

1. **The user-invoked / model-invoked orthogonal axis** is the basis for this project's entire role layering. Only after separating "orchestration" from "discipline" did the structure "managers own orchestration, subagents carry reusable discipline" become possible. This axis is more valuable than any individual skill.
2. **Pushing discipline down into primitives** (`grilling` is the shared substrate of `grill-me` / `grill-with-docs` / `triage` / `wayfinder` / `improve-codebase-architecture`) directly inspired this project: rules like "documents are exclusive" and "evidence is append-only" are written as discipline an agent can reach automatically, rather than scattered across each role's prompt.
3. **Tracer-bullet vertical slicing** changed our dispatch granularity. Previously we sliced horizontally by layer; after switching to "narrow but complete, independently demonstrable, finishable in a single context" paths, the parallelism of `de` and the determinism of acceptance both come from here.
4. **The single-document shape of `to-spec`** (what/why and how in one file) led this project to drop the two documents PRD and design.md, converging deliverables from 3 down to 1; the later document diet (cutting the ADR and the test plan) continued along the same line (`docs/scout/` and `docs/test/evidence/` were later restored as needed, see group-conventions "Document whitelist").
5. **The dual axis of `code-review`** (Spec / Standards) became the basis for separating evidence between `de` self-review and `te` independent review: one skill, two axes, each independent evidence.
6. **The container repo (monorepo of skills) organization** was likewise extracted from his repository practice.

### License and how we use it

The upstream `mattpocock/skills` is under the **MIT License**.

This project **copies none** of his files. There are two handling modes:

- Skill names are referenced as links, reading the source on the upstream `main` branch;
- The **semantics** of `to-spec` and `to-tickets` were rewritten and internalized into this project's own format documents (`pm/spec-format.md`, `dm/templates.md`), with the source noted in the text.

To use the upstream skills themselves, visit [`mattpocock/skills`](https://github.com/mattpocock/skills) directly and follow its MIT license.
