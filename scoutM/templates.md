# ScoutM companion templates (TUI interrogation + se dispatch + output document specification)

> The single full-text carrier; [`SKILL.md`](SKILL.md) keeps a pointer only. Output documents must strictly use the Markdown format of **single-sentence short lines, high-contrast bold, no nested lists**, ensuring the best terminal scrolling experience when viewed through `less`, `glow` or `bat`.

## TUI interrogation interface (render immediately on a selection / architecture / competitive / refactor task)

### Step 1: TUI terminal interactive interrogation

```
   _____                      __
  / ___/_________  __  ______/ /_
  \__ \/ ___/ __ \/ / / / __  / __/
 ___/ / /__/ /_/ / /_/ / /_/ / /_
/____/\___/\____/\__,_/\__,_/\__/
```

```
┌──────────────────────────────────────────────────────────────────────────────┐
│  🤖 [SCOUT CORE] initializing research task...                               │
├──────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  To screen open-source projects precisely and align the architecture,        │
│  please choose the [HIGHEST GUIDING PRINCIPLE] for this project:             │
│                                                                              │
│  [A] ⚡ Performance First                                                     │
│      • Metrics: high concurrency, low latency, memory-frugal, lock-free /    │
│        low-lock design, throughput optimization.                             │
│      • Leans toward: "hardcore" low-level projects battle-tested by big      │
│        tech under high concurrency, plus extreme optimization algorithms.    │
│                                                                              │
│  [B] 🧩 Maintainability First                                                │
│      • Metrics: clear design patterns, high cohesion and low coupling,       │
│        modular layering, friendly to team iteration.                         │
│      • Leans toward: "textbook-grade" open-source references with elegant    │
│        engineering structure and strict interface contracts.                 │
└──────────────────────────────────────────────────────────────────────────────┘
```
👉 Reply [A] or [B] in the terminal, or enter custom weights (e.g. 70% A + 30% B):

### Step 2: Dynamic unfreeze and branch routing

After receiving standard input, unfreeze the task and perform filter routing:

- **[A] Extreme performance track**: automatically append the filter words `benchmark`, `high performance`, `allocation-free`. Focus the audit on memory reuse and async I/O.
- **[B] Readable and extensible track**: automatically append the filter words `clean architecture`, `design patterns`, `extensible`. Focus the audit on interface contracts and decoupled design.

## Output document template (five sections)

### 1. Executive Summary

- **Core decision**: [one sentence stating the final technology stack selection or core architecture decision]
- **Core value**: [state the decisive improvement this decision brings to the current project]

### 2. Current Architecture Gaps

- **Code smells**: [code modules found in the inward research with heavy coupling or bloated implementation]
- **System bottlenecks**: [fatal flaws of the existing system in concurrency, latency or extensibility]

### 3. Technology Trade-off Matrix

| Candidate technology | Maturity | Why selected / eliminated | Preference fit (A/B) |
| :--- | :--- | :--- | :--- |
| **Option one** | Production-grade | [which core trait got it selected] | [fits the preference, state the weight] |
| **Option two** | Incubating | [which historical baggage got it eliminated] | [does not meet the preference metrics] |

### 4. Target Architecture

- **Module layering**: [describe clearly and flatly the responsibilities of each layer's components in the new scheme]
- **Data flow**: [describe the flow path of the core request or data]
- **Design patterns**: [names of the core design patterns used and the problems they solve]

### 5. Phased Roadmap

- **5.1 Key Inspirations**: state exactly which project or competitor, and which concrete mechanism, was referenced.
- **5.2 Roadmap**: list three phases flatly, no nesting; tag implementation items "suggest DM dispatch X".
- **[Phase 1] Core alignment**: change the core data structures or interface signatures. Move closer to the good design. Keep backward compatibility.
- **[Phase 2] Feature evolution**: introduce key optimization algorithms or core abstractions. Localize them and get the core logic running.
- **[Phase 3] Abstraction consolidation**: fully decouple and modularize the functionality. Consolidate it into a general core asset. Produce tests and a canary migration plan.

## se dispatch prompt template

`se` is a subagent generated dynamically by ScoutM at dispatch time and installs no separate skill file: role boundaries, workflow and reporting format are all embedded into the prompt by this template. The name is fixed as `se`; one message fires every call in the same batch. Dispatch is always **background/asynchronous** (mechanism mapping in [`../pm/group-conventions.md`](../pm/group-conventions.md) "Organization"); ScoutM ends the round right after dispatch (return control) and wraps up when woken by the completion notice, never waiting in place.

**Skill passing (hard)**: `se` works with the [`research`](https://raw.githubusercontent.com/mattpocock/skills/main/skills/engineering/research/SKILL.md) skill, and **that address must be written into the prompt**. A subagent does not inherit ScoutM's context; without it, it does not know which research method to use. If the skill's steps conflict with ScoutM's anti-hallucination cross-verification, the skill wins and the two are stacked.

**Dispatch parameters (set on the call, not inside the prompt text)**: `context: "fresh"` (do not inherit the parent conversation) + `timeoutMs` (team discipline item 6: fill it in when writing the ticket, split long chains into reported segments).

```
Role: se, scout-engineer (scout), not ScoutM; no subagent tool, must not dispatch subagents, and when dispatch is needed only write "suggest ScoutM dispatch X". Report only to ScoutM, contact no other role.

Skill: research (https://raw.githubusercontent.com/mattpocock/skills/main/skills/engineering/research/SKILL.md). Research by its method; if this prompt conflicts with it, it wins.

Boundary: research and evidence gathering only; do not change business code, do not write any file to disk (intermediate conclusions included); send conclusions and evidence back in the report message; no external communication, no polling.

Context:
- Timeout: timeoutMs = {milliseconds} (must already be filled at dispatch, never blank, never rely on the default)
- Research question: {one sub-question, one sentence}
- Project root: {absolute path}
- Existing material: {upstream conclusions / related SPEC or plan paths / candidates already ruled out; write "none" if there is none}
- Artifact: **write nothing to disk**. Send your section's conclusions and evidence back to ScoutM, who assembles them and writes the report under docs/scout/ himself
- Citation requirement: performance numbers, version numbers and licences must come from a first-hand official source; every technical claim must be cross-checked against at least 2 independent sources

Workflow:
1. Break the sub-question down per the research skill and fix the search surface
2. Inward: scan the project dependency list and core directories (filter noise such as node_modules / dist), record the current state and constraints
3. Outward: lock 2-3 open-source references and compare the underlying paths (data structures, concurrency model, extension points)
4. Anti-hallucination: trace every conclusion back to a first-hand source; label second-hand retellings
5. Send your section back in the shape ScoutM gave (only your own section, do not merge other people's content)

Reporting format:
Sub-question / artifact path / one-sentence conclusion / key evidence (file:line or first-hand link) / open questions and uncovered items.

Criterion: every conclusion carries a first-hand source; uncovered items are listed explicitly, never dressed up as finished.
```
