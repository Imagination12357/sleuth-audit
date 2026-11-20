# Run state (`state.yaml`)

The authoritative, resumable record lives at `<repo>/.audit/full-audit/<NNN>/state.yaml`; the human-facing report is a derived view. Preserve valid existing records and unknown fields during updates. Use concise YAML, not a transcript of agent reasoning.

This state tracks audit progress and the evidence supporting findings. It does not track fixes or verification of fixes. `complete` describes audit completion; `disproven` means the investigated claim was not a defect under the claimed conditions.

## Naming and lifecycle

- Audit numbers are per repository: `001`, `002`, ...; choose `max(existing numeric run directories) + 1` when starting a **new** run. Never renumber or overwrite a run.
- IDs are `FA-<run>-<finding>` (e.g., `FA-003-012`). For a new finding, use one more than the highest numeric finding suffix in either `findings` or `retired_findings`, starting at `001` when both are empty.
- On resume, retain the run number and IDs. Check whether the source baseline changed since the recorded review. Re-check affected coverage and findings; report unresolved drift. Do not present merged observations from different versions as a single verified snapshot.
- Store `state.yaml` after each review unit or material finding. Generate `report.md` at completion or interruption, including a pause or blocker. Writes within the audit folder do not authorize edits elsewhere in the project.
- `running`: work ongoing. `complete`: every in-scope review unit meaningfully reviewed and each finding assessed or has a documented verification blocker. Findings with inconclusive evidence may remain if their evidence, uncertainty, and any blocker and next check are recorded. `partial`: audit paused with unfinished coverage or finding assessments. `blocked`: that unfinished work cannot continue with current access. A finding's verification blocker alone does not block completion; remaining uncertainty alone does not make a run partial.

## Dirty working tree

In `baseline.note`, list only paths relevant to the audit whose reviewed state differs from `git_head`. Use repository-relative paths. Omit files unchanged from `git_head` and audit-generated records. If there are no relevant differences, omit the path list.

- `modified` applies to a tracked file whose reviewed content differs from `git_head`. Record a content identifier for the bytes reviewed, such as a SHA-256 hash.
- `untracked` applies to a reviewed file Git does not track. Record its content identifier the same way.
- `deleted` applies when the absence of a tracked file matters to the audit. Record the path without a hash.

Example `baseline.note` for an audit with two relevant changes.

```yaml
note: |-
  modified src/parser.rs sha256:2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824
  untracked tests/parser_cases.rs sha256:486ea46224d1bb4fb680f34f7c9ad96a8f24ec88be73ea8e5a6c65260e9cb8a7
```

On resume, compare the recorded paths and content identifiers with the current files and check for new relevant changes. If the earlier contents cannot be identified, disclose the uncertain baseline and re-review affected units.

## Minimal shape

Record the run, status, baseline, scope, coverage, `findings`, and `retired_findings`. Each finding needs an ID, title, location, confidence, severity, condition, impact, and evidence. Use `findings: []` when there are no findings and `retired_findings: {}` when no IDs have been retired. The example below omits optional fields.

```yaml
run: 3
status: partial
baseline:
  git_head: abcdef123456 # null if unavailable
  working_tree: clean # clean | dirty | unknown
scope:
  include: [src, tests, Cargo.toml]
  exclude: [target, .audit, vendor]
coverage:
  - unit: "src/parser"
    status: reviewed # queued | reviewing | reviewed | blocked
  - unit: "src/storage"
    status: queued
findings:
  - id: FA-003-001
    title: "Valid selection rejected after filtering"
    location: "src/parser.rs:88-110"
    confidence: confirmed
    severity: important
    condition: "The user selects B at index 1 in [A, B]; filtering removes A before lookup."
    impact: "Rejects the valid selection of B with an index error."
    evidence: "Static trace: the entry point accepts index 1 for [A, B], filters to [B], then calls get(1) without remapping. get returns None and the caller returns an index error."
retired_findings: {}
```

This example illustrates shape only, not actual audit evidence. `baseline` should capture the available Git revision and the relevant working-tree condition; if Git is unavailable, record another identifiable source version or explicitly state that the baseline is unknown.

## Field semantics

- `coverage`: partition the intended scope into identifiable, non-overlapping logical units. `reviewed` means substantive path and behavior analysis, not a file-list scan. Uninspected or blocked units must remain visible.
- `confidence`: `confirmed | probable | possible | unknown | disproven`.
- `severity`: `critical | important | minor | unknown | null`. `null` is appropriate when a finding is disproven. Do not infer severity from confidence.
- `evidence`: the analysis supporting the finding, checks performed and their results, and any uncertainty or verification blockers. Include command and result when a test was actually run. Keep this concise; do not store raw logs or copied source files.
- When resuming older records, merge any legacy `verification` value into `evidence`, preserving distinct information without repeating it. Use only `evidence` in new or updated findings.
- One underlying defect gets one entry in `findings` within a run. A repeated observation in a later run may cite a prior ID in prose. Keep each audit tied to its recorded source baseline.

## Retired finding IDs

- When merging a duplicate, keep useful evidence in the surviving finding. In the same checkpoint, remove the duplicate from `findings` and add its ID to `retired_findings` with `state: merged` and `merged_into` set to the surviving ID. The target must be in `findings` in the same run at the time of the merge.
- If a finding record is deleted, move its ID to `retired_findings` with `state: removed`. Keep retired IDs in that map for the lifetime of the run. Keep disproven findings and their evidence in `findings`.
- When looking up a retired ID, follow `merged_into` until reaching a finding or an entry with `state: removed`. Keep the recorded links when a merge target is later merged or removed. If a link is missing or the chain repeats an ID, report the original reference as unresolved.
- Older states without `retired_findings` may have lost IDs before this rule existed. Before issuing a new ID in such a run, reconstruct the retired IDs from available records. For an ID known to have been issued whose record is gone, use `state: removed` if its merge target cannot be recovered. If the issued IDs cannot be established, issue new IDs in a new run.

In this example, looking up `FA-003-003` reaches `FA-003-001` through `FA-003-002`. The next new ID is `FA-003-005`.

```yaml
retired_findings:
  FA-003-003:
    state: merged
    merged_into: FA-003-002
  FA-003-002:
    state: merged
    merged_into: FA-003-001
  FA-003-004:
    state: removed
```

## Optional fields

Add these only when applicable. Omit empty optional values; do not fill every finding with placeholders.

- `note` on `baseline` or a coverage unit: context that changes how the baseline or review status should be interpreted, including source drift or a blocked review unit's reason.
- If a relevant check was unavailable or skipped and has no associated finding, use `coverage[].note` to record the check, why it was not run, and what remains unverified. Add the note even when `findings` is empty. For a check spanning several units, record it once under a relevant unit and name the affected scope.
- `next` on a finding: the outstanding verification step. Include it when there is an actionable follow-up; if no check can proceed, explain the blocker in `evidence`.
- `history`: short changes in classification with reasons, **only when reclassified**. Do not store private deliberations, raw command logs, or copied source files.
- `parent`: optional ID of the finding whose **investigation led to this finding**. It is provenance, not causality. Derive children by looking up `parent`; do not store a redundant `children` field.
- `related_to`: optional IDs of **distinct** related findings. This is an undirected, untyped association; do not imply `causes`, `requires`, or `duplicates`. Record a pair once, on either endpoint, and treat it as symmetric when reporting. Avoid self-links. Cross-run references are allowed when the referenced run is available.

### Example: an unconfirmed finding

In this unconfirmed version of the fictional example above, `confidence` and `evidence` change and `next` records the remaining check:

```yaml
confidence: probable
evidence: "The caller checks the index before filtering, but the lookup helper's index-remapping behavior has not yet been inspected."
next: "Inspect the lookup helper to determine whether it remaps the original index."
```


Keep the state sufficient for a future agent to resume without reading previous chat history. Persist key scope and reproduction evidence, not every exploratory step.
