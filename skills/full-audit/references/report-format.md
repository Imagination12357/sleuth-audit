# Audit report (`report.md`)

Generate `<repo>/.audit/full-audit/<NNN>/report.md` from `state.yaml`. Keep the report self-contained, scannable, and grounded in source analysis and verification evidence. Write for a developer who needs to verify and act on the findings: use direct sentences, concrete conditions, and plain language. Cut filler, exaggerated claims, generic praise, and repetitive summaries. Preserve uncertainty when the evidence is incomplete. Findings are not fixes; never change code to make the report look complete.

## Classification

Full Audit uses the severity labels from the Superpowers code-review template, including the parenthetical wording. Superpowers also reviews style, architecture, test gaps, and other improvements. Full Audit limits findings to correctness defects and defines severity by their impact, so its definitions differ from the original template.

Minor retains the label "Nice to Have" for compatibility and covers low-impact defects here. Unknown is an additional value for findings whose impact cannot yet be judged. See [third-party notices](../THIRD_PARTY_NOTICES.md) for the source and MIT license.

| Severity | Meaning for full-audit |
| --- | --- |
| **Critical** (Must Fix) | Credible consequence is severe: e.g. major data loss/corruption, major security breach, or system-wide failure. |
| **Important** (Should Fix) | Material correctness failure or meaningful functional/contract breakage under reachable conditions. |
| **Minor** (Nice to Have) | Real but low-impact correctness defect; **not** style, documentation polish, or optional refactoring. |
| **Unknown** | Impact cannot currently be judged. |

Assess severity by the impact if the defect occurs, independently of confidence, frequency, or ease of exploitation. A `Critical / Possible` finding is valid. A disproven finding has no severity (`null`).

| Confidence | Minimum basis |
| --- | --- |
| **Confirmed** | A reproducing observation **or** a complete, logically supported static proof with reachable conditions and no unresolved counterexample. |
| **Probable** | Strong evidence for a real defect, but a material validation gap remains. |
| **Possible** | Plausible failure path whose preconditions or outcome have not been established. |
| **Unknown** | Not enough information to judge even the existence of the proposed defect. |
| **Disproven** | Investigated and shown not to be a defect under the claimed conditions. |

Do not label a guess `Confirmed` because a line looks suspicious or a test suite passed. A failed test unrelated to the claim does not confirm it. Avoid numerical probabilities without calibration.

## Report outline

Generate the report at completion or interruption, including a pause or blocker. Use the run status from `state.yaml`: Complete, Partial, or Blocked. Running is used only in state while work continues; refresh the report when that work completes or is interrupted.

```markdown
# Full Audit — <project> — FA-<NNN>

- Status: Complete | Partial | Blocked
- Baseline: <Git revision / dirty state / any source drift>
- Scope: <reviewed paths and important exclusions>

## Summary
<Counts by severity and confidence, key risks, and limitations>

## Findings
### FA-<NNN>-<MMM> — <short title>
- Severity / Confidence: <value> / <value>
- Location: `<path>:<line range>` (or explain if unavailable)
- Conditions: <concrete reachable scenario>
- Failure and impact: <expected vs actual behavior>
- Evidence: <analysis supporting the finding, checks and results, uncertainty or verification blockers>
- Relationships: <parent and related_to IDs, when present>
- Follow-up: <smallest useful next check, if needed>

## Disproven leads
<IDs and concise reasons; do not silently omit them>

## Coverage and limitations
<Reviewed units, unreviewed/blocked units, excluded paths, unavailable tests,
 source drift, and other reasons the audit might miss defects>
```

Order active findings primarily by severity (Critical, Important, Minor, Unknown), then confidence and ease of verification. Summaries must not hide `unknown`, `possible`, or blocked findings. Put detailed, disproven entries in a short section rather than inflating the actionable list.

Use `findings` for finding counts and lists. When a relationship cites an ID in `retired_findings`, keep the cited ID visible and explain where the chain ends. Follow the [state schema](state-schema.md#retired-finding-ids) to distinguish a surviving finding, a removed target, or an unresolved reference.

In **Coverage and limitations**, include the unavailable or skipped checks recorded in `coverage[].note`, including when there are no findings. Preserve the reason each check was not run and what remains unverified.

## Evidence rules

- Point to precise file locations and the relevant call path or invariant. Cite source positions accurate for the recorded baseline; if line numbers drift, update or flag them.
- Explain the minimal triggering condition and **expected versus actual** effect. Separate observations from assumptions and proposed reproductions.
- State command, target, and outcome for tests actually executed. Summarize logs; do not paste excessive tool output or fabricate environment access.
- When validation reveals a new independent defect, report its own ID and investigation provenance; simple consequences belong in the original issue.
- State findings as conditional where appropriate. Neither “no findings” nor `complete` proves the codebase is bug-free.
