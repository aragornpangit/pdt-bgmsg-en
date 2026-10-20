# SPEC Format (PM)

Read this reference only when writing a SPEC: template, granularity prohibition, writing principles, version-bump rules. Naming and ownership are in [`SKILL.md`](SKILL.md) and [`group-conventions.md`](group-conventions.md).

The SPEC is **the product department's only deliverable document** and the starting point of the whole chain: it carries both what/why (the problem and the solution, from the user's perspective) and how (implementation decisions and test decisions), so that DM and TM can start work directly. The format is internalized from mattpocock [`to-spec`](https://raw.githubusercontent.com/mattpocock/skills/main/skills/engineering/to-spec/SKILL.md) and does not depend on external commands. **80-line baseline, may moderately exceed** (if it does, cut redundancy first; if nothing more can be cut, record the reason in the file header).

## Template

```markdown
# SPEC: {feature title}

- Number: {seq} | Version: v1 | Written: {YYYY-MM-DD HH:MM:SS}
- Preceding SPEC: {related number, delete this line if none}

## 1. Problem Statement

{Problem statement from the user's perspective. Cite a report number or file:line; do not write empty phrases like "there are some problems"}

## 2. Solution

{Solution from the user's perspective: what is to be done this time}

## 3. User Stories

Numbered list, format `As an <actor>, I want a <feature>, so that <benefit>`; be as exhaustive as possible, covering every aspect of the feature.

## 4. Implementation Decisions

Modules and interfaces, technical clarifications, architecture decisions, Schema changes, API contracts, key interactions. One per line.

## 5. Testing Decisions

What counts as a good test (**test only external behavior, not implementation details**), which modules will be tested, existing precedents for similar tests.

## 6. Out of Scope

{Items explicitly excluded}

## 7. Further Notes

Root cause analysis (file:line), change history, runtime verification items, dependencies and constraints. Delete the whole section if there is nothing.
```

## Granularity prohibition

**Do not write concrete file paths or code snippets** (`They may end up being outdated very quickly`).

The only exception: snippets produced by a prototype that encode a decision more precisely than prose (state machine / reducer / schema / type shape), which may be inlined under the corresponding decision with a note that it comes from the prototype. Cut to the decision-dense part, not a whole demo.

## Writing principles

1. **Problems are evidenced**: the counterexample is "there are currently some problems"; replace it with a report number or `file:line`
2. **User stories are exhaustive**: section 3 is the body of the SPEC, not decoration
3. **Decisions are executable**: section 4 goes down to modules, interfaces, and contracts so that DM does not have to guess
4. **Tests are testable**: section 5 states the criterion for "test only external behavior"
5. **Non-goals are declared**: section 6 stops scope creep before work starts

## Terminology

Use the naming already present in the project's `CONTEXT.md` (or domain glossary) and in the code; when neither exists, fix it according to the code naming and define it once where it first appears in the SPEC. The SPEC's wording matches the code, so DM and TM do not invent a second set of names.

## Version-bump rules

**Small step updates within a version modify the current file**, without renaming:

1. Read the current version and confirm which items need changing
2. Append the change history to Further Notes (round / parameter values / feedback, with the latest round in bold)
3. Update the related parts of the solution and acceptance

**Substantive revisions (the problem, solution, or scope is replaced wholesale, or a convention is changed by arbitration) bump the version into a new file** (naming in "File naming" of [`group-conventions.md`](group-conventions.md)):

1. The new file gets version number `v{N+1}`, the timestamp of the new write, and the same `{seq}`
2. Mark the old file's header in place as "superseded by vX" (a trace vs. a residue: the old wording is explicitly voided, not deleted to pretend it never happened)

## Disposable after use

The SPEC records the decisions already made in this piece of work, for the next context window to read, and is no longer maintained after the work is delivered. There is only one place for knowledge that must be kept long term: domain terms go into `CONTEXT.md` (maintained by PM).
