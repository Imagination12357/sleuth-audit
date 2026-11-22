---
name: sleuth-audit
description: Use when the user explicitly requests an audit of an existing repository or selected paths for latent correctness defects. Excludes ordinary PR/diff reviews, feature work, and debugging one known failure.
---

# Sleuth Audit

Audit repositories for latent correctness defects, assess severity and confidence separately, and save resumable YAML checkpoints and Markdown reports.

Review the whole repository by default, or the paths selected by the user. Account for every in-scope review unit and prioritize findings supported by evidence.

## Guardrails

- Honor user scope and repository instructions. Otherwise include first-party source, tests, configuration, build logic, and integration boundaries; exclude generated outputs, caches, vendored code, and `.audit/`. Disclose material exclusions.
- Never edit application files, tests, manifests, Git history, or `.gitignore`. Store persistent records only in `.audit/sleuth-audit/<NNN>/`; use isolated temporary files for safe reproductions. Do not install dependencies without permission.
- Run relevant existing tools and safe targeted tests when useful. Obtain approval before destructive, external-effect, or unexpectedly costly operations. Record unavailable or unrun checks, not imagined results.
- Review the current codebase, not just Git changes. Use Git only to establish and monitor the audited baseline.

## Procedure

1. **Initialize.** Read [state-schema.md](references/state-schema.md). Determine root, scope, baseline, and logical review units. Resume the requested run or allocate the next project-local index without overwriting a run.
2. **Explore.** Map entry points, interfaces, invariants, state changes, error paths, and cross-module interactions. Track coverage in `state.yaml`. A file read or passing test alone is **not** a completed review.
3. **Triage.** Record distinct plausible defects with immutable `SA-<NNN>-<MMM>` IDs, confidence, and impact-based severity. Ignore style, optional refactoring, and unsupported speculation.
4. **Validate.** Trace reachable conditions to observable consequences; actively test counterarguments, guards, and contracts. Reproduce safely where useful. A complete static proof may also confirm a bug. Record the analysis, checks, results, and remaining uncertainty together in the finding's `evidence`. Record relevant unavailable or skipped checks without an associated finding in `coverage[].note`. Never invent experiments or line references.
5. **Reclassify.** Update conclusions with brief evidence. Investigate newly discovered independent defects under new IDs. Preserve merged or deleted IDs in `retired_findings` as specified in [state-schema.md](references/state-schema.md). `parent` is investigation provenance; undirected `related_to` denotes association only, never proven causation.
6. **Persist.** Checkpoint after each review unit or material finding. At completion or interruption, consult [report-format.md](references/report-format.md), generate `report.md` in direct, evidence-led language, and distinguish confirmed defects, unresolved suspicions, disproven leads, and unreviewed scope.

## Classification and exit

- Confidence: `confirmed | probable | possible | unknown | disproven`. `unknown` means insufficient evidence to judge. Preserve disproven leads briefly.
- Severity uses the Superpowers labels `critical | important | minor`, with definitions scoped to correctness defects; `unknown` if impact is unclear; `null` if disproven. Judge impact if triggered, independently of confidence or frequency. See [report-format.md](references/report-format.md) for definitions and attribution.
- Mark `complete` only after every in-scope unit is meaningfully reviewed and each finding is assessed or has a documented verification blocker. Findings with inconclusive evidence may remain if their evidence, uncertainty, and any blocker and next check are recorded.
- Use `partial` while coverage or finding assessments remain unfinished and the audit is paused; use `blocked` when that work cannot continue with current access. A finding's verification blocker alone does not block completion. See [state-schema.md](references/state-schema.md) for status definitions. If the baseline changed, revalidate affected findings or disclose drift. Never assert that a completed audit proves the absence of bugs.
- Stop unproductive follow-up chains with recorded uncertainty. Keep code fixes in a separate task after saving the audit records; authorization to fix a bug does not change this audit's source-editing prohibition.

## References

- [State schema](references/state-schema.md): read when creating, resuming, or updating audit state; owns YAML fields and ID rules.
- [Report format](references/report-format.md): read when classifying findings or writing the report; owns severity definitions and evidence standards.
