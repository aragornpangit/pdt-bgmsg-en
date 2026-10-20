# ScoutM companion templates (TUI interrogation + output document specification)

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
- **5.2 Roadmap**: list three phases flatly, no nesting.
- **[Phase 1] Core alignment**: change the core data structures or interface signatures. Move closer to the good design. Keep backward compatibility.
- **[Phase 2] Feature evolution**: introduce key optimization algorithms or core abstractions. Localize them and get the core logic running.
- **[Phase 3] Abstraction consolidation**: fully decouple and modularize the functionality. Consolidate it into a general core asset. Produce tests and a canary migration plan.
