# Test department (test) templates (TM)

**One requirement, one file**: `docs/test/report/{spec-seq}-testReport-{semantic-slug}-{timestamp}-v{N}.md` (naming see [`../pm/group-conventions.md`](../pm/group-conventions.md) "file naming", `{spec-seq}` reuses the SPEC's id), **cases and results live in the same file**. **Update in place within a version** (appending cases does not rename); a substantive revision bumps the version into a new file. **40 lines baseline, may moderately exceed** (if exceeded, delete redundancy first; when nothing can be deleted, register the reason in the file header).

There is no separate test plan document: cases go straight into the case table of the report. Evidence has two layers: conclusion evidence is inlined in the table as `file:line`; host-side machine-check evidence (`selftest` logs, machine-check output) goes into the repo at `docs/test/evidence/{spec-seq}-testReport-{semantic-slug}-ev/` (restored, for TM archiving only; one directory per seq, a report version bump does not change the directory).

## Template

```markdown
# Test report: {feature title}

- Number: {spec-seq} | Version: v{N} | Related SPEC: `docs/pd/spec/{spec-seq}-spec-*.md`
- Related development plan: `docs/dev/plan/{spec-seq}-devPlan-*.md` | Build gate: {result}

## Cases and results

| Case ID | Category | Review item/input | Expected | Actual | Execution | Result | Evidence |
|---|---|---|---|---|---|---|---|
| XX-CODE-01 | CODE | | | | te#1 | PASS | file:line |

**The "Execution" column is mandatory** (this is exactly what the machine check in TM self-check item 3 looks at): fill in the dispatched batch (`te#<batch>`); if TM genuinely did it himself, fill in `TM(reason)`. Criterion-level conclusions must be traceable from this column to the `te` receipt.

## Problems found

| Level | Description | Location | Suggestion |
|---|---|---|---|

**Overall verdict**: {pass / conditional pass / fail} + one-sentence reason (items not verified at runtime are marked here)
```

**Sections may be deleted**: once the case table covers every acceptance criterion there is no need for a separate coverage section; when "Problems found" is empty, **delete the whole section** (group convention: do not write a "none" placeholder). **Exceeding the baseline: delete redundancy first; when nothing can be deleted, register the reason in the file header**.

## Case categories (`XX` customized per SPEC topic, the only list)

`XX-CODE` code review (Spec + Standards dual axis), `XX-VAL` logic, `XX-UI` interaction, `XX-REG` regression, `XX-EDGE` boundary.

## te dispatch prompt template

`te` is a subagent generated dynamically when TM dispatches, with no standalone skill file: the role boundary, verification means and reporting format are all embedded into the prompt by this template. The name is always `te`; start all calls in the batch in one message. Dispatch is always **background/asynchronous** (mechanism mapping see [`../pm/group-conventions.md`](../pm/group-conventions.md) "Organization"), TM ends the round as soon as it sends (return control), and is woken by the completion notice to review again; it does not wait in place for results. Case categories as above, same name and meaning as on the TM report side.

**Dispatch parameters (written on the call, not inside the prompt text)**: `context: "fresh"` (do not inherit the parent conversation) + `timeoutMs` (team discipline item 6: fill it in when writing the cases; device tickets / long chains >= 45 min and report in segments). The template body also lists `timeoutMs` as a line so TM does not miss it when copying the ticket.

```
Role: te, test execution agent, not TM; no subagent tool, must not dispatch subagents. Reports to TM only, does not contact pm/dm or other te agents.

Boundary: review and verify only, modifying any business code is forbidden (during defect diagnosis add test / reproduction scripts only); do not change the test report; evidence is inlined in the return message, do not write any file to disk (host-side machine-check evidence that needs to be kept is archived by TM into `docs/test/evidence/{spec-seq}-testReport-{semantic-slug}-ev/`).

Context:
- Timeout: timeoutMs = {milliseconds} (must already be filled at dispatch, no blanks, no relying on defaults)
- SPEC: {spec-path}
- Development plan and progress: {plan-path}

Cases in this batch ({category}):
| Case ID | Review item | Expected result |
|---|---|---|

Means: Read the source, grep static assertions, code walkthrough (substitute inputs one by one and analyze branch outputs).
- `XX-CODE` cases use the code-review dual axis (Spec axis: consistency of the implementation with the SPEC; Standards axis: compliance with repo standards), skill text: https://raw.githubusercontent.com/mattpocock/skills/main/skills/engineering/code-review/SKILL.md
- Hard defects and performance regressions use the diagnosing-bugs diagnosis loop (the red-green loop adds test / reproduction scripts only, never changes product code), skill text: https://raw.githubusercontent.com/mattpocock/skills/main/skills/engineering/diagnosing-bugs/SKILL.md
- Every result must give file:line or a grep hit / zero-match as evidence; a conclusion without evidence is treated as BLOCKED
- A reported defect must carry three things: the original error text + file:line + the reproduction command or log path (team discipline item 12; without them TM cannot send it back to the development department)
- On finding a P0 / regression / red-line violation, report to TM immediately

Return strictly in the following format, do not add anything else:

## Conclusion: PASS / FAIL / BLOCKED

| Case ID | Expected | Actual | Result | Evidence |
|---|---|---|---|---|

## Problems found
| Level | Description | Location | Suggestion |
|---|---|---|---|
```
