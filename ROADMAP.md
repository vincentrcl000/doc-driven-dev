<!--
  Purpose: Evolution Roadmap for the doc-driven-dev skill.
  Function: Records future enhancement plans, categorized by priority.
  Last Modified: 2026-05-06 12:15:00 (CST)
-->
# doc-driven-dev — Evolution Roadmap

> This document records the evolution plan for the skill itself. Features already implemented are listed in `SKILL.md`. This document tracks increments that have been evaluated but not yet implemented.

## Version History

| Version | Date | Key Changes |
|:---|:---|:---|
| 1.0 | Initial | Adoption / Plan / Design / Develop / Sync five-stage closed loop. |
| 1.1 | 2026-05-05 | Idempotent refresh + Goal-driven + Step 1.5 code probe + L1/L2/L3 multi-model routing + Concurrent CAS + Rollback + Authoring Checkpoints + Template versioning + Glossary + `depends_on` graph constraints. |
| 2026.5.6 | 2026-05-06 | **Open Source Generalization**: Removed specific project hardcodings; generalized project profile examples; path sanitization and placeholders; restructured README for various Agent environments. |

Current Version: **2026.5.6**

---

## Evaluated Increments (Planned Delivery in Batches)

Categorized by priority; each item includes "Motivation / Trigger / Affected Files / Risks."

### P1 (Next Milestone, 1.2 Candidate)

#### 1. Doc/Code Migration Tracking (Rename Detector)
- **Motivation**: Inventory becomes `stale` when users rename files.
- **Trigger**: During Stage 0 Path R scan, perform auto-rebinding for stale entries using `git log --follow` + content SHA fingerprints.
- **Affected Files**: `workflows/00-adoption.md §Path R`, `schema/project-schema.md` (new `inventory[].content_fingerprint`), new `rename_detection` check in `workflows/05-sync.md`.
- **Risks**: Degradation if Git is unavailable; performance needs testing on large repos.

#### 2. CI / Pre-commit Integration
- **Motivation**: Stage 4 only triggers in-session; drifts are easily missed otherwise.
- **Trigger**: User says "Integrate sync into CI" or "Add pre-commit hook."
- **Affected Files**: New `scripts/run-sync.{sh,ps1}`, new `hooks/pre-commit` template, new "Automation hooks" section in `SKILL.md`.
- **Risks**: Script differences across CI platforms; failure threshold policy (block vs. warn) requires user configuration.

#### 3. Version Archive/Migration Workflow
- **Motivation**: No workflow for `scope` migration during V10 → V11 transitions.
- **Trigger**: Auto-prompt when user says "Proceed to new version" or `scope.active_versions` needs change.
- **Affected Files**: New `workflows/10-version-bump.md`.
- **Risks**: Downgrade rules for maintenance/archived versions need discussion.

### P2 (Engineering Polish, 1.3 Candidate)

#### 4. Document Categorization for Non-code Artifacts
- **Motivation**: Sync reports false positives for ADRs or OPS manuals where `code_paths` are naturally empty.
- **Implementation**: Add `inventory[].kind: "code_driven" | "artifact" | "adr"`; `05-sync.md` selects check types by kind.
- **Affected**: Schema field expansion, sync routing table.

#### 5. AskUserQuestion Batching
- **Motivation**: Intensive questioning in adoption/planning leads to UX fatigue.
- **Implementation**: Provide "One-time Form" templates (`schema/adoption-form.md`) for pre-filling and bulk submission.
- **Affected**: `workflows/00-adoption.md`, new `templates/adoption-form.md`.

#### 6. Large Project Performance
- **Motivation**: Full sync is slow for 100+ documents.
- **Implementation**:
  - `inventory[].last_scanned_mtime` field.
  - Default incremental scanning for full sync (skip by comparing mtime).
  - `--full` parameter to force full scan.
- **Affected**: `05-sync.md`.

#### 7. Privacy / Local Override Config
- **Motivation**: Internal paths in `config.json.references.local_repos` should not be committed.
- **Implementation**: Support `{METADATA_DIR}/config.local.json` (gitignored by default), merged into `config.json` with priority at runtime.
- **Affected**: `SKILL.md §Configuration & State`, split writes during adoption.

#### 8. Failure Modes and Observability
- **Motivation**: No unified handling for Grep timeouts or incomplete LLM returns.
- **Implementation**: Add "Failure Classification + Retry Strategy" sections to each Step; add `errors[]` to `state.json`.
- **Affected**: All workflows.

### P3 (Quality Metrics & Reports, 1.4+)

#### 9. Knowledge Capture for Drift Fixes
- **Motivation**: `drifts[].fixed=true` loses the "how it was fixed" context.
- **Implementation**: Expand to `drifts[].fix_record: { method, prevention_rule, related_drifts[] }`.
- **Usage**: Stage 2 retrieval of prevention rules from similar drifts to proactively avoid issues in new docs.

#### 10. Document Coverage Metrics
- **Motivation**: Quantify document completeness of the project.
- **Implementation**: Mode 4 "coverage report" in `workflows/05-sync.md`: calculate the percentage of files in the codebase not mapped to `inventory[].code_paths[]`.
- **Output**: Markdown / HTML reports.

#### 11. Dry-run / Simulation Mode
- **Motivation**: Preview all writes the skill is about to perform.
- **Implementation**: Global `--dry-run` parameter; all Write/Edit actions are redirected to log prints.
- **Affected**: `SKILL.md` frontmatter, "Dry-run Behavior" sections added to all workflows.

#### 12. Configurable Scoring Thresholds
- **Implementation**: `config.json.quality_thresholds: { min_total: 8, min_accuracy: 7, ... }`.
- **Impact**: Convergence criteria / self-audit checklists.

#### 13. Automatic Doc Aging Detection
- **Implementation**: Add `age_check` to Stage 4 sync; suggest `stale` if `last_updated` exceeds N months.
- **Config**: `config.json.sync.stale_threshold_days`.

#### 14. Cross-project Inventory References
- **Motivation**: Shared design patterns between AwesomeApp and peer-system can be cross-referenced.
- **Implementation**: `schema.peer_projects[].cross_refs[]`; currently for declaration only, no cross-project sync scanning.
- **Risks**: Heavy path dependency.

#### 15. Report Export
- **Implementation**: `workflows/05-sync.md` supports `--export md|html|json` to generate independent drift report files.

#### 16. i18n Multi-language Documents
- **Postponement Reason**: Current user scenarios are mono-lingual; no strong demand yet.
- **Preparation**: `schema.glossary[].translations[]` field placeholder reserved.

---

## External Capabilities Not in Scope

The following capabilities are handled by dedicated tools; the skill only consumes their output:

| Capability | Responsible Tool |
|:---|:---|
| Multi-doc Batch Orchestration | `oh-my-claudecode` / `oh-my-codex` |
| Git Operations | Manual User / IDE Integration |
| CI Scheduling | GitHub Actions / GitLab CI |
| Code Indexing/Search | ripgrep / IDE Built-in search |

---

## Feedback Channel

To report a new gap, directly edit the `## Evaluated` section of this document to append an entry, ranked by priority. Also, register a drift in `state.json.drifts[]` (type: `skill_enhancement`) linked to the discovery.
