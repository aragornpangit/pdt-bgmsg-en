# AGENTS.md: the organizational constitution of this repo (release mirror)

This repo is the **English edition (pdt-en) of the pdt-group role skill family**, not a skill container repo. The Chinese original is [aragornpangit/pdt-group](https://github.com/aragornpangit/pdt-group); content and rules are the same, only the language differs. The container repo (internal private repo `Skills`, layout `skills/pdt-group/<role>/`) is the only place where writing happens; this repo is synced one-way from it, for external distribution and reading.

Differences from the container repo's constitution: the container repo's `skills/<bucket>/<name>/` structural rules, §8 structural checks, and multi-bucket registration flow **do not apply** here (this repo uses a flattened layout). This repo has no machine-check scripts; human reading plus container-repo validation are the gate.

## 1. Skeleton (flattened layout: one directory per role, directly at the repo root)

```
.
├── AGENTS.md          # this file
├── README.md          # human-facing table of contents and installation notes
├── ACKNOWLEDGMENTS.md # acknowledgments to upstream mattpocock/skills
├── LICENSE            # MIT
└── pm/  dm/  tm/  scoutM/   # four installable roles, each with SKILL.md and companion reference docs
```

**The repo holds only four role directories**: no more `docs/` (human-facing copies of role descriptions), `scripts/` (machine-check scripts), or `.gitlab-ci.yml` (internal CI); all three were deleted on 2026-10-20 by user decision and must not be recreated.

**No `SKILL.md` at the root**; `SKILL.md` may only appear one level down, at `<role>/SKILL.md`.

## 2. Roles and content ownership

- `pm` / `dm` / `tm` / `scoutM` are the four installable roles; `de` / `te` are subagents generated dynamically by DM / TM at the dispatch stage and **install no skill files** (templates are embedded in `dm/templates.md` and `tm/templates.md`). There are two further on-demand derivations that likewise install no skill files: PM's evidence-gathering / verification subagents, and ScoutM's se (reconnaissance engineers, several dispatched on demand)
- **Single source of group conventions = `pm/group-conventions.md`**: organization, communication, document whitelist, expression discipline and the team discipline table are written once, only there; the four roles' SKILL.md files keep only pointers and their role-specific execution surface, and **do not restate the full text** (prevent drift across copies)
- **Single detailed edition of expression discipline = `pm/communication-style.md`**: the jargon table and grep self-check exist only in that file
- To change a discipline, change group-conventions.md first, then check that each SKILL.md's pointers and execution-surface description are still accurate
- `ACKNOWLEDGMENTS.md` and the container repo's `skills/pdt-group/pm/inventory.md` are **skill lists living in two separate repos**; changing one requires syncing the other (same-named upstream paths must match, and every skill cited in the ACK must be registered in the ledger)

## 3. Hard requirements

1. **No em-dash (U+2014) in prose**. Scope: all `.md` files and code comments. To break a sentence, rewrite it (comma / colon / period / parentheses / conjunction); blind character substitution is forbidden.
2. **The directory name must equal the `name` field of the `SKILL.md` in that directory**.
3. **Relative links must resolve**.
4. **No dead code left behind**: when content is retired, delete it; do not build a `deprecated/` graveyard; leave a pointer elsewhere if needed.
5. **No credentials in the repo**: no file may contain a plaintext password / token / PAT; examples always use environment variable placeholders.
6. **One-way sync direction**: container repo → this repo. Direct edits here are overwritten on the next sync; when a change is truly needed, change the container repo first and then sync (an urgent fix may land here first, but must be registered for backflow).
7. **SKILL.md must not exceed 50 lines**: details and long lists go in companion reference docs in the same directory (such as `disciplines.md` / `templates.md`); SKILL.md keeps only the entry, core constraints and pointers. The line caps for deliverables (SPEC / plan / report / research) are a **baseline** and may be moderately exceeded (see group-conventions "Writing discipline").
8. **Role descriptions live only in `<role>/SKILL.md`**: no separate four-section human-facing copy, and `README.md` does not restate the organization or document discipline already in group-conventions, only a one-line pointer.
