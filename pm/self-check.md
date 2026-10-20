# PM Self-Check and Arbitration Rules

> **PM must follow these himself** (not only require them of the team).
> The **single full text** of the general discipline (the three-part reporting set, verifying the numbers when announcing completion, verification target = delivery target, how to write discipline and the admission threshold) is the "Team discipline" table in [`group-conventions.md`](group-conventions.md); this file keeps only PM-specific content and does not restate it.

## 1. When verification is mandatory (rework-cost criterion)

**Clause**: **any action where "getting it wrong once = redoing a whole round" must be verified, without exception**.

**Criterion**: **rework cost** (calculable), **not "important / unimportant"** (which relies on feeling).

**Typical examples (non-exhaustive)**: the code base point cited when writing a ticket; build artifacts; numbers reported externally.

**Reason**: importance judgments depend on people's awareness; rework cost **can be answered on the spot** (ask "if this is wrong, do we have to redo it?"), **and does not need the prior self-awareness that "this is important" to trigger**.

## 2. Arbitration principles (PM only)

| Principle | Companion action | Reason |
|---|---|---|
| **Downgrade / retire / defer ⇒ must carry a restore condition** | State "under what conditions it is restored"; otherwise it counts as a **permanent exemption** (not allowed) | A downgrade with no restore condition means that six months later nobody knows whether it was a decision or an omission |
| **If the assumptions behind a criterion are refuted by measurement ⇒ change the criterion** (not the implementation), and **the original design intent must be preserved** | Record the original intent in one sentence; the revised criterion must explicitly state what it now guards against | Changing only the implementation and not the criterion ⇒ the next person judges by the old criterion and gets it wrong again |
| **Thresholds must not be adjusted in order to pass** | Any change must note "basis = measurement"; **extrapolation does not change the verdict** (extrapolation is inference, not measurement) | Adjusting a threshold to pass = turning a measurement into a wish |
| **Indirect evidence must not substitute for direct evidence** | For any statement that "a resource / capability is available", **probe it on the spot before sending and attach the output** | When indirect and direct evidence conflict there is no error report, only silent distortion |
| **Verify evidence sufficiency before changing a delivered value** | Give the effective sample count **and** the variation range across samples; **the evidence standard is not lowered because it is cheap or reversible** | Low cost makes people lower the evidence bar, and this is the most common source of missed judgments |
| **Conclusions from a bench / simulated environment must not be labeled "measured"** | Items involving real-environment semantics may only be labeled "inferred" or "to be verified"; when a simulated environment conflicts with the real one, **suspect the simulated environment first** | A simulated environment differs from the real one in some semantics ⇒ the conclusion is distorted and no error is reported |

## 3. Communication

**Pointer**: [`communication-style.md`](communication-style.md) (jargon table, grep self-check, the two exceptions). **Every outward statement goes through there.**
