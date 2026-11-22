# Sleuth Audit

Audit repositories for latent correctness defects, assess severity and confidence separately, and save resumable YAML checkpoints and Markdown reports.

Sleuth Audit reviews the current codebase using code analysis and safe reproductions, and records what it could and could not verify. By default, it reviews the whole repository; you can limit the scope to selected paths. It writes audit records under `.audit/sleuth-audit/<NNN>/` in the repository being audited.

## Install

Sleuth Audit provides one skill for Codex and Claude Code.

### Claude Code

```bash
claude plugin marketplace add Imagination12357/sleuth-audit
claude plugin install sleuth-audit@sleuth-audit-marketplace
```

Then invoke `/sleuth-audit:sleuth-audit` in a Claude Code session.

### Codex

```bash
codex plugin marketplace add Imagination12357/sleuth-audit
```

Open the plugin browser in a supported Codex client and install/enable **sleuth-audit** from **sleuth-audit-marketplace**. Registering a marketplace does not install its plugins.

If you only want the skill and do not need marketplace installation, copy `skills/sleuth-audit/` into the skill directory for your agent.

## Usage

In Codex, ask:

```text
Use $sleuth-audit to audit this repository.
```

To limit the scope or resume an earlier run:

```text
Use $sleuth-audit to audit only src/parser and its tests.
Use $sleuth-audit to resume run 001.
```

In Claude Code, use `/sleuth-audit:sleuth-audit` with the same requests. When resuming, Sleuth Audit checks for source changes and rechecks affected coverage and findings.

## What it does

1. Maps the selected codebase, entry points, invariants, and review units.
2. Records review coverage and potential bugs with stable IDs.
3. Traces how suspected failures could occur, checks for guards and counterexamples, and tests each claim through code analysis or safe reproduction.
4. Records which defects it confirmed, which leads need more evidence, and which claims it ruled out.
5. Saves the state and a readable report; unfinished audits can be resumed.

An audit is complete when every in-scope review unit has been reviewed and each finding has been assessed or has a documented verification blocker. Findings with inconclusive evidence can remain in a completed audit. Completion does not guarantee that every defect was found.

## Confidence

Confidence records how strongly the evidence supports each finding. Sleuth Audit confirms a defect by reproducing it or proving from the code that the failure can occur. The report also records what remains uncertain and summarizes claims the audit ruled out.

## Severity

Severity describes the impact if a defect occurs. Sleuth Audit uses the Superpowers labels Critical (Must Fix), Important (Should Fix), and Minor (Nice to Have). It records severity as Unknown when the impact is unclear.

A suspected bug that could cause major data loss can be marked `Critical / Possible` while the audit is still checking whether the failure can occur. See the [classification rules](skills/sleuth-audit/references/report-format.md#classification) for all levels.

## Output

For run `001`, the audited repository receives:

```text
.audit/sleuth-audit/001/
├── state.yaml
└── report.md
```

`state.yaml` tracks scope, coverage, findings, and verification status. Sleuth Audit updates it after each review unit or material finding and generates `report.md` at completion or interruption. The report explains the evidence for each finding and what remains unverified. It also summarizes disproven leads. New runs use `002`, `003`, and so on; resuming a run keeps its number.

## Safety and scope

Sleuth Audit leaves source files, tests, manifests, Git history, and `.gitignore` untouched. It does not install dependencies without permission or run destructive, external-effect, or unexpectedly expensive operations without approval. Generated files, caches, vendored code, and earlier `.audit/` runs are excluded by default unless the requested scope says otherwise.

Style suggestions, documentation polish, and optional refactoring are outside the audit scope. Bug fixes belong in a separate task after the audit records have been saved. Sleuth Audit does not track whether findings have been fixed or their fixes verified.

## Source layout

- `skills/sleuth-audit/SKILL.md`: audit workflow and constraints
- `skills/sleuth-audit/references/state-schema.md`: audit state and finding ID format
- `skills/sleuth-audit/references/report-format.md`: findings, severity, and evidence rules
- `skills/sleuth-audit/agents/openai.yaml`: Codex skill presentation metadata
- `plugin.json`: portable Agent Plugins manifest for Codex
- `.claude-plugin/`: Claude Code manifest and marketplace
- `.agents/plugins/marketplace.json`: Codex marketplace
- `THIRD_PARTY_NOTICES.md`: Superpowers attribution and full MIT license
- `skills/sleuth-audit/LICENSE` and `skills/sleuth-audit/THIRD_PARTY_NOTICES.md`: copies of the project license and third-party notices for standalone skill distribution; keep them identical to the root files

## License and credits

This project is licensed under the [MIT License](LICENSE). The severity labels come from [Superpowers](https://github.com/obra/superpowers); see [third-party notices](THIRD_PARTY_NOTICES.md) for its copyright notice and full MIT license.

The [no-ai-slop](https://github.com/petergyang/no-ai-slop/blob/main/skills/no-ai-slop/SKILL.md) editing guidelines informed the README and report wording. Neither reference is a runtime dependency.
