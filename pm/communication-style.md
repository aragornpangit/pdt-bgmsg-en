# Communication Style (group expression conventions, single detailed version)

> This file is the **single detailed version** of the group-wide "expression" discipline (jargon mapping table + grep self-check + two exceptions). For the discipline clauses, see "Expression (all roles)" in [`group-conventions.md`](group-conventions.md). Each role's SKILL.md no longer maintains its own mapping table.
> Source: user instruction: **"Use less software engineering jargon, switch to a simple, clear style, so that an engineer who knows a little software engineering can understand it"**; additional requirement: **"Talking, writing files, writing comments: all should use less jargon, be simple and clear"** (scope expanded from speech/messages to everything written down).
> Scope: all expression aimed at the **user** + dispatch and arbitration conclusions aimed at the **team** (DM / TM / de / te) + **all text written into the repository** (SPEC, CONTEXT.md, commit messages, annotation notes, and other files) + **code/script comments**.

## 1. Reader profile

**"An engineer who knows a little software engineering"**, not a compiler, and not a product novice.

- He knows what "interface", "test", "log", "compile" mean;
- He does **not** know shorthand our team invented internally ("convention", "range", "has teeth" ...);
- The test, in one sentence: **if I left the company, could a new engineer understand this paragraph directly?**

## 2. Five rules

| # | Rule | Counter-example -> Good example |
|---|---|---|
| 1 | **State the conclusion and the impact first, then the details** | ❌"Criterion 6's `max/min` is 1.5539, over threshold" -> ✅"Frame smoothness is unstable (measured fluctuation 1.55x, the target is no more than 1.25x). The picture now speeds up and slows down" |
| 2 | **The first time a term appears it must be turned into plain language** (put the term in parentheses, so files are easy to match) | ❌"This convention needs re-measuring on the same build" -> ✅"This number has to be measured again on the same version of the program (the team calls it 're-measure on the same build')" |
| 3 | **Use numbers and concrete consequences**, not abstract judgments | ❌"Coverage is low, quality needs improvement" -> ✅"40 of 100 test cases failed" |
| 4 | **Use a table for multi-item comparisons**, not long paragraphs | See conventions |
| 5 | **Reports to the user are 10 lines or fewer**, details go into files (give them when the user asks) | - |

## 3. Jargon replacement table

> Usage: **say the right column first; add the left-column term when you need to match against files.**

| Internal term | How to say it to users / newcomers |
|---|---|
| criterion | acceptance standard / what this check verifies |
| convention | how it is computed / which number to use |
| range | the scope this check covers (**where it reaches and where it does not**)|
| gather evidence / re-measure | get the data / measure it again |
| write to disk | save as a file |
| pre-gate | the checks before work starts (no start if they fail)|
| red line | places you must not touch |
| has teeth / negative arm | the test really catches the problem / deliberately introduce an error and see whether the test reports it |
| mutation testing | deliberately break one piece of code and see whether the test raises an alarm |
| fail-closed | if the data is insufficient it counts as fail (rather than not judging, do not let it pass)|
| silent failure | something went wrong but nothing was said |
| idempotent / atomic / race | repeated execution gives the same result / either do it all or do none of it / both sides rush to change the same data |
| deadlock | each waits for the other, stuck forever |
| orthogonal | the two things do not affect each other |
| converge / align | reach agreement / unify what both sides say |
| wrap up | settle it, finish |
| closed loop | accounted for, has a result, traceable |
| gate | check (stop if it fails) |
| ledger / reconcile | count verification |
| raise back | restore to strict checking |
| trigger surface | under what conditions this criterion takes effect |
| anchor / anchor point | pin down which copy, which version |
| write down | put it in writing |
| attribute | find the cause |
| stratify / group | count separately, do not mix them into one number |
| baseline | the reference point before improvement |
| self-contained | readers understand it without opening other files |
| whitelist | only these few are allowed, nothing else |
| vertical slice | split off a narrow piece of functionality that runs end to end and can be verified independently |
| synergy / leverage / best practice / at scale | (**never use**, say concretely what to do)|
| bump the version / final sign-off | update the requirement document / final signature |
| explicit dormancy | not used for now, but **not deleted** (state under what conditions it is re-enabled)|
| fallback | the backup approach when something goes wrong |
| canary / instrumentation / event logging | small-scope trial / add statistics / log events |
| desensitize | hide sensitive information |
| carrier | where it is written / which file it lives in |
| contract | the input/output format both sides agreed on |
| mapping | one-to-one correspondence |
| granularity | how finely it is split |
| invariant | a rule that must not be broken under any circumstances |
| water level | how far current progress or capacity has reached |

## 4. Supporting actions (**must happen first**, not "remember to pay attention")

> Based on the team meta-discipline: **a discipline that only says "what not to do" is incomplete; it must come with supporting actions.**

**Self-check before sending (machine-checkable)**: run the draft through the word list below; any hit means rewrite.

```bash
# Usage: save the content to be sent as a file, then run; expected output is empty
# Word list = column 1 of the mapping table in §3, word by word (`/`-separated words are all included); the two tables must be kept in sync by hand (change one side and you must change the other)
grep -niEw 'criterion|convention|range|gather evidence|re-measure|write to disk|pre-gate|red line|has teeth|negative arm|mutation testing|fail-closed|silent failure|idempotent|atomic|race|deadlock|orthogonal|converge|align|wrap up|closed loop|gate|ledger|reconcile|raise back|trigger surface|anchor|anchor point|write down|attribute|stratify|group|baseline|self-contained|whitelist|vertical slice|synergy|leverage|best practice|at scale|bump the version|final sign-off|explicit dormancy|fallback|canary|instrumentation|event logging|desensitize|carrier|contract|mapping|granularity|invariant|water level' <draft file>
```

**Two exceptions (a hit may be kept)**:

1. **Real names in code** (file names, variable names, commands, metric names): **must be kept as-is**, but **must be immediately followed by a plain-language explanation**. Example: keep the key `roi_pairs_truncated_n`, then add "(the number of entries whose outward-expansion ratio data was truncated)".
2. **In dispatch to DM / TM**, since both sides have established shared understanding, precise terms may be kept, but **do not pile up more than three in the same sentence**, and **the conclusion sentence still uses plain language**.

**Self-check declaration**: for reports to the user, run the `grep` above once before sending, and **report the hit count as 0 (or list the retained items and the reason)**. Per team discipline #2 (self-checks and reported numbers must actually run), **reported numbers must come from an actual run, not from memory**.

**Writing files and comments**: when writing a SPEC / report draft / commit message / code comment, also run through the §3 mapping table: **write plain language first, add the term when you need to match against files**; comments explain "why it is done this way", and do not pile up abbreviations and internal nicknames. The judgment standard is unchanged: **a new engineer can understand without asking anyone**.
