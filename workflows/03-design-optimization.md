<!--
  Purpose: Workflow for Stage 2.2 — Design Document Optimization.
  Function: Elevates design quality through reference research, gap analysis, and multi-model refinement. Includes 9-step deep path and batch orchestration (omc/omx).
  Last Modified: 2026-05-06 11:00:00 (CST)
-->
# Stage 2.2 — Design Document Optimization

> Purpose: **Improve the actual solution quality of design documents through reference project research, gap analysis, and multi-model refinement** until they converge to a "no flaws can be found" state. This is NOT just a document formatting cleanup.

## §0 Depth Selection Gate (Mandatory Pre-condition)

**Before any "Optimize / Polish / Refine / Improve Quality" request is processed**, the skill must first confirm the depth type with the user:

| Depth | Use Case | Output | Entry |
|:---:|:---|:---|:---|
| **deep (Default)** | Need to discover design space, perform gap analysis, refactor solutions, and cross-model refine; all optimization with "improve design quality" semantics goes here. | Reference research report + Gap analysis matrix + New draft + Six-dimensional evaluation + Target convergence criteria met. | §4 Standard Deep Path (9 Steps) |
| **batch-deep** | ≥ 5 documents requiring systematic deep optimization. | Same as deep, executed in parallel. | §5 omc / omx Batch (Injection Spec) |
| **light** (Explicitly stated) | Formatting / Terminology / Numbering / Status markers / Pseudocode cleanup — **Explicitly NOT improving solution quality**. | Draft with the same format. | §2 Light Consistency Cleanup |
| **audit-only** | Only perform diagnostic scoring without editing documents. | Scoring report + Issue list. | See `05-sync.md` Mode 3 |

**Ambiguity Resolution**:

- User says "Optimize / Polish / Refine / Multi-round research / Autoresearch / Improve quality / Benchmarking" → `deep`.
- User says "Clean up pseudocode / Unify numbering / Fix formatting / Align terminology / Quick pass" → `light`.
- User says "Evaluate / Score / Check consistency / Diagnose" → `audit-only`.
- **Default (No clear signal) → deep**. The skill should respond with "Proceeding with deep optimization (Step 1 Reference Research → Step 9 Final Convergence); if only formatting cleanup is needed, please switch to light."

**Pre-audit Documents (e.g., `doc-optimization-plan.md`)**: **Can only be used as known issue inputs for Step 6 (Six-dimensional Evaluation)**, not as a concluding framework for "optimization ends here." Pre-audit scoring dimensions (e.g., 4 dimensions) never replace the 6-dimensional evaluation in the deep process.

Write the selected type to `state.review_session[].mode`. Default is `deep`.

---

## §1 Common Prerequisites (Input Gating)

> Regardless of depth, run input gating before execution. However, in `deep` mode, `reference_projects`, `benchmark_docs`, and `constraints` **must be non-empty** (if `none`, user must explicitly declare and accept the consequences); in `light` mode, these fields are optional.

**Mandatory Gate**: The following 4 categories of input must be collected before entering any path in §2 / §4 / §5. If missing, use `AskUserQuestion` proactively; do not skip. Run this for every new optimization session (reuse previous values from `state.json.review_session[].inputs` as defaults).

### 1.1 Required Input List

| # | Field | Meaning | Action if Missing | Persistence Location |
|:---:|:---|:---|:---|:---|
| 1 | **goal_kind** | Fix accuracy / Remove pseudocode / Align with reference / Complete structure (Multi-select) | AskUserQuestion, select at least 1. | `state.review_session[].inputs.goal_kind[]` |
| 2 | **reference_projects** | List of reference project paths (e.g., `/path/to/peer-project`, `/path/to/industry-standard`) | AskUserQuestion: "Use / to separate; enter `none` if no reference." If `none`, record for audit (will affect final score). | `config.references.local_repos[]` + current `inputs.reference_projects[]` |
| 3 | **benchmark_docs** | List of benchmark doc paths (Auto-recommend from project `document_inventory[]` or manual) | Auto-recommend candidates from inventory per §1.7 protocol (P1→P6 priority); verify existence if manual; if `none`, final score cap drops to 7/10. | `inputs.benchmark_docs[]` |
| 4 | **required_sections** | Mandatory deep sections (e.g., **Implementation Strategy (Non-code)** / Edge Cases / Integration Points / Output Handling / Execution Mode / Configuration Model / Implementation Phases / Error Handling). See §1.6 "Morphological Constraints of Deep Sections." | AskUserQuestion; Default = Set of subsections actually contained in benchmark docs (Auto-extracted via Grep); user can toggle/edit. | `inputs.required_sections[]` |
| 5 | **constraints** | Current constraint list: Auto-load all `project_constraints[]` + user-added session-specific constraints (e.g., "Only align with PTC, no sandbox this time"). | Auto-load project level; AskUserQuestion for session-level; user says `none` if no additions. | `inputs.constraints[]` (contains `from_project[]` and `session_only[]`) |
| 6 | **roadmap_doc** (Optional but recommended) | Path to Gap Analysis / Evolution Roadmap. | Not mandatory, but auto-suggested if `roadmap_references[]` exists in schema. | `inputs.roadmap_doc` |
| 7 | **review_level** (Optional) | Expected level L1 / L2 / L3. | If not specified, auto-determined by `multi-model-review.md §0.2`. | `inputs.review_level` |

### 1.2 Collection Protocol

Before entering §2 / §4 / §5, the skill **must output a briefing** for user confirmation/revision, e.g.:

```
🎯 Input Confirmation for Single-Doc Optimization

  Goal (goal_kind)           : Align with reference + Complete structure
  Reference (reference_projects): /path/to/peer-project
                                /path/to/industry-standard
  Benchmarks (benchmark_docs) : [Inventory recommendations, 1-3 docs per §1.7 priority]
    ① docs/phases/phase-a-xxx.md  (score 9.0, status=approved)       ← P1 Primary
    ② docs/phases/phase-b-yyy.md  (Adopted doc, implementation_status=done)  ← P4 brownfield
    ③ Manual specify / none
  Required Sections (required_sections): Implementation Strategy(Non-code) / Edge Cases /
                                Integration Points / Output Handling / Execution Mode /
                                Config Model / Phases / Error Handling
  Constraints (constraints)
    Project-level (from_project): PC-001 AwesomeApp is Enterprise Server, not IDE proxy.
                                PC-002 Reference industry-standard/peer-system for concepts only.
                                PC-003 No Plugin System replication.
    Session-level (session_only): [Need to add? e.g., Align PTC only, no sandbox]
  Roadmap (roadmap_doc)       : Docs/.../Gap_Roadmap_vN.0.md
  Review Level (review_level) : Auto-determine (Current models → L?)

Proceed with these inputs? (yes / modify item / cancel)
```

### 1.3 Reuse & History

- `inputs` from the last session for the same `doc_id` are used as default suggestions.
- Docs in the same phase or batch reuse the same set of inputs to avoid fatigue.
- After the user says `yes`, all fields are written to `review_session[].inputs` for next use.
- Any `omc/omx` batch call uses these fields to fill the injection block (§5.3 Spec Injection).

### 1.4 Fallback Logic

If the user does not respond (e.g., auto-mode), downgrade according to priority:

1. Read the most recent `inputs` for the same `doc_id` from `state.json.review_session[]`.
2. Read `config.references.local_repos[]` as `reference_projects`.
3. If still missing, explicitly mark `reference_projects=[]` / `benchmark_docs=[]`. The final self-check "References marked as conceptual" will fail, triggering rework.

### 1.5 Other Prerequisites

1. Read recent drift reports from `state.json.drifts[]` as a known issue list.
2. Check inventory item `references[].type=prior_audit`; if exists, auto-load that external audit section.

### 1.6 Morphological Constraints for Deep Sections (Anti-Pseudocode Alignment)

> **Key Clarification**: Deep sections require "Design Decisions at the implementation level," **NOT the implementation code itself**. Code in design documents easily drifts from the actual codebase and causes ambiguity. This is a HARD RULE, consistent with `design-refine.md §Rule 1 / §Hard Constraint #1`.

All sections with "Implementation" semantics in `inputs.required_sections[]` (e.g., `Implementation Strategy`, `Key Tech Details`, `Core Component Design`) MUST be expressed in the following **non-code forms**:

| Form | Description | When to Use |
|:---|:---|:---|
| **Decision Table** | "Decision / Choice / Reason" columns; can add "Alternative / Rejection Reason." | Algorithm selection, concurrency primitives, data layout, error propagation. |
| **Module Map** | "Target Responsibility / Reuse Existing Module (with path) / New Module (with target slice)." | Implementation path, avoid re-inventing existing logic. |
| **Interface Contract Snippet** | Type definitions / Trait signatures / Function signatures, **≤ 10 lines per block**, for contract description only. | Cross-module boundaries, external API shape. |
| **Data Flow / State Machine** | Mermaid or ASCII; draw states and transitions only, no implementation. | Complex control flow, life cycles. |
| **Error Matrix** | "Error Source / Detection Point / Recovery Action / Residual Risk" columns. | Fault tolerance, degradation, rollback paths. |
| **Edge Case Checklist** | Bulleted list; each item has a trigger condition + mitigation strategy. | Boundaries, race conditions, resource exhaustion. |

**Redlines** (Rework triggered if violated, per `design-refine.md §Self-check #2 / §Hard Constraint #1`):

- Single pseudocode/code block > 10 lines → Must replace with decision table or module map.
- Implementation blocks like `fn xxx() { /* implementation */ }` (even if ≤ 10 lines) → Keep signature only if used as "Interface Contract"; function body MUST be deleted.
- Appearance of crates / types / paths not yet introduced in the project → Treated as "Creating Code"; see Hard Constraints #1 / #2.
- Chapter titles with **language names** like "Rust Implementation / Python Implementation" are considered anti-patterns; rename to "Implementation Strategy" / "Key Implementation Decisions" / "Implementation Mapping."

**Depth Interpretation** (Resolving ambiguity between "Depth" and "Anti-pseudocode"):

- Depth ≠ Code Volume. Depth = Decision Density + Boundary Coverage + Precise Mapping to existing code.
- A qualified "Implementation Strategy" section contains **no function bodies** yet allows implementers to uniquely translate decisions into code.
- If it's "impossible to explain a mechanism without writing code," it's a granularity issue; split components, add state machine diagrams, or add decision tables instead of reverting to pasting code.

### 1.7 benchmark_docs Candidate Protocol

> Adapts to both **Greenfield** (New project, docs reviewed with score) and **Brownfield Adoption** (Existing docs, score unknown).

#### Discovery Steps

1. Read `project-schema.json.document_inventory[]` (from adoption).
2. Determine the `phase` of the **current target doc** (from metadata or filename).
3. Filter per the priority table below; take the first non-empty result, up to 3 docs.

#### Priority Table

| Priority | Filter Criteria | Use Case |
|:---:|:---|:---|
| P1 | `status=approved` AND `score≥8` AND `phase` = current doc phase | Greenfield, same-phase high-quality benchmark. |
| P2 | `status=approved` AND `score≥8` (Any phase) | Greenfield, cross-phase high-quality benchmark. |
| P3 | `status=approved` AND `score≥7` (Any phase) | Greenfield, second-best choice. |
| P4 | `implementation_status=done` (`score` is null) | **Brownfield Adoption**, finalized docs as structural benchmarks. |
| P5 | `implementation_status=partial` AND `phase` = current doc phase | Brownfield Adoption, minimum viable benchmark. |
| P6 | No matches | `AskUserQuestion` for manual path or `none`. |

#### Display Format (Used in §1.2 Briefing)

| Source | Format |
|:---|:---|
| Reviewed Doc (Score known) | `{Relative Path} (score {N}.{M}, status=approved)` |
| Adopted Doc (Score null) | `{Relative Path} (Adopted, implementation_status={done/partial})` |
| Manually Specified | `{Relative Path} (Manual)` |

#### Constraints

- Maximum of 3 recommendations; if inventory candidates are insufficient, append "③ Manual specify / none."
- User can: Select from list / add manual path / delete item / answer `none`.
- If user answers `none`, record `benchmark_docs=[]`; final score cap drops to 7/10.
- **Adopted docs are used for structure and chapter completeness ONLY**; when `score` is null, they do not participate in "Score target met" logic, only for `required_sections` skeleton extraction.

---

## §2 Light Consistency Cleanup (Mode=light ONLY)

> ⚠️ **WARNING**: This path **DOES NOT improve design quality**. It only performs formatting / terminology / numbering / status / pseudocode cleanup. If the user said "Optimize / Polish / Refine / Multi-round" but entered here, the §0 depth selection was wrong; go back to §0 immediately.

> Prerequisite: Completed §1 Input Gating and received `inputs` object; `mode` must be `light`.

### Step 1 — Audit Status
1. `Read` target doc full text.
2. `Grep` corresponding code areas to generate an "Implementation Status Summary" (to correct outdated descriptions).
3. Load entries from `state.drifts[]` related to this doc as a known issue list.

### Step 2 — Feed Refine-Prompt
Use `prompts/design-refine.md` as the sub-LLM system prompt, providing:
- Target doc full text.
- Project template placeholder values.
- Implementation status summary (from Step 1.2).
- Known issue list (from Step 1.3).

**Note**: This step does NOT feed reference project takeaways (that's the `deep` mode's job), does not perform gap analysis, and does not suggest solution refactoring.

### Step 3 — Per-Chapter Rule Application
Strictly follow the 7 rules in `refine-prompt` for each chapter:
1. Identify and delete misleading pseudocode.
2. Replace with decision tables.
3. Layer components / risks / acceptance.
4. Clear discrepancy recording.
5. Four-category file planning.
6. Numbering consistency.
7. Language and formatting.

### Step 4 — Self-Check
Run `refine-prompt` §Self-Check Checklist (12 items); return to Step 3 if any fail.

### Step 5 — Update Metadata
- Bump `version`.
- Update `last_updated`.
- Update `score` (if possible).
- Advance `status` to `approved` or keep as `in_review` as appropriate.

---

## §3 Integrating Reference Benchmarking

If the optimization involves reference project benchmarking (most common scenario):

1. Use `prompts/reference-extraction.md` to extract implementation details from reference projects.
2. Produce a "Benchmark Comparison Table": Reference approach vs. Current solution vs. Adoption suggestions.
3. When optimizing the doc, note "Conceptual Reference / Direct Adoption" in the §0.2 Reference Table.
4. Guard against "Over-engineering": If the reference approach exceeds current project scale, mark as "Postponed" in §0.6 Implementation Redlines.

**Anti-Over-engineering Principles**:
- "Nice-to-have" capabilities from reference → Postpone to later slices.
- Core mechanisms from reference → Adopt, but prune to minimum viable for the current project.
- Heavy infrastructure from reference (if project lacks it) → Seek lightweight alternatives or postpone.

---

## §4 Standard Deep Path (Mode=deep Default Entry)

> **The essence of deep optimization is NOT "letting multiple models look at it," but "Reference Research → Gap Analysis → Solution Refactoring → Multi-model Refinement." Multi-model is just a means to achieve refinement; even with L1 single-model downgrade, NOT ONE of the 9 steps can be skipped.**

See the full 9-step process in `prompts/multi-model-review.md`. **Must run `multi-model-review.md §Step 0` first** for capability probing and routing (L1 / L2 / L3 determination).

### §4.0 9-Step Minimum Workload (Mandatory for L1/L2/L3)

| Step | Mandatory Action | Skippable Condition |
|:---:|:---|:---|
| 1 | For each `inputs.reference_projects[]`, use `prompts/reference-extraction.md` to produce a "Reference Implementation Report" (with Adoption Level table filtered by constraints). | Skippable if `reference_projects=none` and user explicitly waives, but final score cap drops to 7/10. |
| 2 | Based on Step 1 + `inputs.benchmark_docs[]` depth, **draft a new version by chapters** (or majorly revise existing draft). Cover `inputs.required_sections[]`. | NOT skippable. |
| 3 | Check if chapter depth matches benchmark (Implementation Strategy / Edge Cases / Integration / Output / Execution / Config / Phases / Error Handling). Deepen using §1.6 Morphological Constraints. **Deepening output is LIMITED to Decision Tables / Module Maps / Interface Contracts ≤ 10 lines / Data Flows / Error Matrices / Edge Case Checklists; do NOT "add depth" by adding pseudocode**. | NOT skippable. |
| 4 | Model 2 independently drafts a comparison solution (Downgraded to "Red-team Persona" in L1). | NOT skippable; for L1, use Red-team persona + session clear. |
| 5 | Comparison & Merger (or verify discrepancies after user rewrite). | NOT skippable. |
| 6 | Model 3 **Six-dimensional Holistic Evaluation**: Gap alignment / Solution completeness / Rigor in current environment / Design consistency / Adoption rationality / Anti-over-engineering. | NOT skippable. **These 6 dimensions are the criteria for deep scoring, NOT the 4 dimensions of pre-audit docs**. |
| 7 | Adoption decisions for each suggestion (Categorized A/B/C/D). | NOT skippable. |
| 8 | Post-revision Verification (**incl. Change Propagation Check**: if any decision was flipped, signature changed, or config added/removed in Step 2-7, must grep all references throughout the document and sync each one; also verify all fenced code blocks are correctly paired). | NOT skippable. |
| 9 | Final Convergence (L3: 2 consecutive ✅ rounds; L2: 1 ✅ round + User confirmation; L1: Mandatory human final review). | NOT skippable. |

**Core Cycle**:

```
Step 1 Research ──▶ Step 2-3 Draft/Deepen ──▶ Step 4 Independent Draft ──▶ Step 5 Merge
    ▲                                                             │
    │                                                             ▼
    └─── Return to Step 2 if split ◀── Step 9 Converge ◀── Step 8 Verify ◀── Step 7 Adopt ◀── Step 6 6D Eval
```

### §4.1 Pre-trigger Check

| Condition | Action |
|:---|:---|
| `config.json.models[]` not initialized | Run `multi-model-review.md §0.1` to guide user in declaring available models. |
| Only 1 model available | Auto-downgrade to L1 (Multi-persona + Red-team + Mandatory human final review). **L1 does NOT waive any Step, only the "True Independent Model" guarantee.** |
| ≥ 2 models available but user refuses to switch | Downgrade to L1; mark `declined_switch: true` in `state.json.review_session`. |
| Model switch command fails (User error) | Abort current Step; skill must not "pretend to be another model" using the current one. |

### §4.2 Per-Step Model Switch Verification

Before entering Step 4 / Step 6, the skill MUST output the switch command specified in `multi-model-review.md §0.3` and wait for user `ready` confirmation. If the user does not respond or says "cannot switch," downgrade to L1 or abort per §4.1.

### §4.3 Relationship with Pre-audit Documents

If a pre-audit document (e.g., `doc-optimization-plan.md`) exists in the schema:

- ✅ **Load** as "Known Issue Input" before Step 1 starts.
- ✅ **Reference** its diagnostic items during Step 6 (6D Evaluation).
- ❌ Do NOT **replace** the 6D evaluation of §4.0 with its scoring system (e.g., 4 dimensions).
- ❌ Do NOT **skip** Step 1 Reference Research because the pre-audit marked it as "Covered."

Use case: All optimization with "Improve Quality" semantics goes through this path. Especially for critical architecture decisions, cross-module interfaces, and security/performance critical paths.

---

### §4.6 Output Write-back (Single-Doc Mode Closure, **Mandatory**)

> **Core Constraint**: After §2 light mode Step 5 / §4 deep mode Step 9 final convergence passes, **all** write-back actions in this section MUST be executed. Any missing item means this optimization is incomplete — do NOT mark `done`.
> Equivalent to §5.6 batch write-back; they share the same write targets, differing only in trigger timing (single doc vs. batch completion).

#### pre-writeback Gate (Mandatory — all must pass before write-back)

> If ANY of the following 3 checks fails, **do NOT proceed to write-back**; return to Step 8 to fix, then re-run the gate:

| # | Check Item | Pass Condition |
|:---:|:---|:---|
| G-1 | **Change Propagation Complete** | If this session flipped any decision / changed a signature / added/removed config: all references to the changed item (body text, design rationale, summary tables, example code, common pitfalls, etc.) have been grepped and synced — no residual contradictions anywhere in the document |
| G-2 | **Format Integrity** | All fenced code blocks in the document have correctly matched opening/closing ` ``` `; no markdown content (headings / tables / body text) is accidentally wrapped inside a code block |
| G-3 | **Config Consistency** | If config keys were added/removed/renamed: all config tables, JSON examples, and `Default` columns in the document are in sync; no orphaned config rows remain |

After all gates pass, output one confirmation line:
```
pre-writeback gate: G-1 ✅ / G-2 ✅ / G-3 ✅ (or N/A)
```
Then proceed to the write-back matrix below.

#### Write-back Target Matrix

| # | Write Target | Content | Trigger Mode |
|:---:|:---|:---|:---:|
| 1 | `project-schema.json.document_inventory[doc_id].score` | Write current scoring object (see score structure below) | light + deep |
| 2 | `project-schema.json.document_inventory[doc_id].status` | Advance to `approved` (final ✅) or keep `in_review` (final ⚠️) | light + deep |
| 3 | `project-schema.json.document_inventory[doc_id].last_updated` | Today's date | light + deep |
| 4 | `project-schema.json.revision` | `+1` | light + deep |
| 5 | `<docs_root>/design-plan.md §1.2 Score Table` | Score cells, status column, total score updated for this doc | light + deep |
| 6 | `<docs_root>/design-plan.md §4 Progress Tracking` | Status and completion date updated for this doc | light + deep |
| 7 | `<docs_root>/design-plan.md §5 Execution Log` | Append `[YYYY-MM-DD] {doc-id} {mode} optimization done, score {old}→{new}` | light + deep |
| 8 | `state.json.review_session[].outputs` | Score snapshot + final verdict + adoption suggestion list | deep (required) / light (optional) |
| 9 | Target design doc metadata (header or sidecar) | Bump `version` / update `last_updated` / `score` (synced with #1) | light + deep |

#### score Write Structure (schema `document_inventory[*].score`)

Supports two parallel scoring dimensions (fill per what was actually produced this session; leave null if not assessed):

```json
{
  "quality": {
    "total": 8.5,
    "accuracy": 9,
    "completeness": 8,
    "actionability": 9,
    "consistency": 8,
    "scored_at": "2026-05-06",
    "scored_by": "design-refine"
  },
  "deep_review": {
    "total": 8.7,
    "gap_alignment": 9,
    "solution_completeness": 9,
    "rigor_on_current_env": 8,
    "consistency_with_existing": 9,
    "reference_adoption_balance": 8,
    "anti_overdesign": 9,
    "scored_at": "2026-05-06",
    "scored_by": "multi-model-review-L2",
    "review_session_id": "rs-20260506-0001"
  }
}
```

| Dimension Set | When to Write | Source |
|:---|:---|:---|
| `quality` (4D) | §2 light mode self-check passed / §1 Step 2 initial scan / §4 Step 6 L1 downgrade path | `prompts/design-refine.md` self-check list |
| `deep_review` (6D) | §4 deep mode Step 6 complete (L1/L2/L3 all write) | `prompts/multi-model-review.md §Step 6` |

#### Write Order & Atomicity

1. Write `state.json.review_session[].outputs` first (most easily rollback-able).
2. Then write the document's own metadata (bump version).
3. Then write `project-schema.json` atomically (score + status + last_updated + revision all in one write).
4. Finally write `<docs_root>/design-plan.md` (human-readable dashboard, written last to avoid half-done state being visible).

If any step fails, the skill must output a rollback notice — do NOT leave the doc half-`done` and half-`in_review`.

#### Verification

After write-back completes, output the confirmation table:

```
✅ Single-doc optimization write-back complete — {doc-id}
  • schema.document_inventory.score   : quality {N}.{M} / deep_review {N}.{M}
  • schema.document_inventory.status  : {old} → {new}
  • design-plan §1.2 Score Table       : refreshed
  • design-plan §4 Progress Tracking   : status {old} → {new}
  • design-plan §5 Execution Log       : +1 entry
  • state.json.review_session         : rs-{id} outputs written
  • doc metadata                      : version {old} → {new}
```

Failure to output this table means write-back is incomplete.

#### §2 Light Mode Step 5 Absorbed Here

`§2 Step 5 — Update metadata` "update score (if possible)" is upgraded to: must execute all write-back matrix items for light mode (targets #1/#2/#3/#5/#6/#7/#9 required, #8 optional).

---

## §5 External Batch Orchestration (omc / omx)

### §5.0 Host-aware Selection

Select the corresponding batch orchestrator based on `config.json.host`:

| host | Primary Commands | Private Metadata Dir (NOT managed by skill) |
|:---|:---|:---|
| `claude-code` | `/oh-my-claudecode:*` | `.omc/` |
| `codex` | `/oh-my-codex:*` | `.omx/` |
| `generic` (Windsurf, etc.) | Skip §5, use §4 Multi-model or §2 Single-doc. | — |

The following command examples use `oh-my-claudecode`; for Codex hosts, replace with `oh-my-codex` (parameters share the same semantics). The skill only passes parameters and consumes output; it **does NOT read/write their private metadata directories**.

### §5.1 Trigger Conditions

Suggest batch mode if any of the following are met:
- ≥ 5 documents to be optimized in the design plan.
- Manual optimization of one round is complete; need batch convergence.
- User explicitly says "Batch / Autopilot / Team authoring."

### §5.2 Command Mapping

| Scenario | Command | Core Inputs |
|:---|:---|:---|
| Batch authoring new docs | `/oh-my-claudecode:team` | Design plan path + Benchmark path + Reference path. |
| Batch optimizing existing docs | `/oh-my-claudecode:autopilot` | Doc list + Convergence direction. |

### §5.3 Input Convention (Spec Injection Version)

**Core Principle**: omc / omx are independent orchestrators; they do NOT read our `templates/` / `prompts/` / `schema`. All specifications MUST be **injected inline** into the prompt so the batch process can self-verify. Do not rely on the skill to fix issues afterward.

#### §5.3.0 Prompt Construction (Skill-automated)

Before initiating an omc/omx call, the skill **MUST** collect and assemble the "Spec Injection Block" from these sources:

| Injected Content | Source File | Form |
|:---|:---|:---|
| Chapter Skeleton | `templates/design-doc.md` | §0-§8 Titles + One-line requirement per section. |
| Metadata Fields | `templates/doc-metadata-schema.md` §Mandatory + §Optional | Field list + Example. |
| Self-Check Checklist | `prompts/design-refine.md §Self-check` | Full 12 items. |
| Glossary | `project-schema.json.glossary[]` | `term / synonyms / definition` table. |
| **Project Constraints** | `project-schema.json.project_constraints[]` + current `inputs.constraints.session_only[]` | ID + Statement + Category, as inviolable boundaries. |
| Adoption Rules | `prompts/reference-extraction.md` "Conceptual vs. Direct" | 3 Adoption criteria. |
| Rework Triggers | `design-refine.md §Hard Constraints` | Full 6 redlines. |

Once assembled, feed into omc/omx as an **additional fenced block** following the main command. The user can review the full prompt but does not need to manually organize it.

#### §5.3.1 team Command (Batch Authoring New Docs)

```
/oh-my-claudecode:team

Start team authoring for remaining design docs based on [design-plan.md]:
- Roles: Architect (Drafter) + Reviewer (Quality) + Benchmarker (Reference Comparison).
- Scope: [List of docs from design-plan.md §2.2].
- References: [Path 1] [Path 2].
- Principles:
  1. Comply with gap alignment and evolution roadmap in [design-plan.md].
  2. Solution completeness and consistency with current code.
  3. Reference gap analysis and anti-over-engineering.
- Autoresearch Convergence Criteria: Must pass all 12 self-check items in the "Spec Injection Block" below.
- Rework if any doc fails; do not block other documents.

====== Spec Injection Block (Skill Generated, DO NOT delete/edit) ======

[Chapter Skeleton (Must be in order, rework if missing)]
§0 Implementation Status
§0.1 Phase Slicing Plan   — Slice table + Status + Core content.
§0.2 Reference Table      — Conceptual Reference / Direct Adoption categories.
§0.3 Discrepancy Log      (Required if divergence exists).
§1 Background & Mission
§2 Layered Architecture   — Component table divided by "Implemented / Planned."
§3 Core Components       — Decision Tables (Decision/Choice/Reason), NO pseudocode > 10 lines.
§4 Implementation Phases
§5 Key Tech Details
§6 Key Risks            — Status markers (✅/⚠️/🔜) + target slice for each.
§7 Acceptance Criteria    — Completed [x] / Planned [ ], each verifiable.
§8 File Planning         — Four categories: Created / Planned New / Planned Mod / Not Introduced.

[Metadata Header (Mandatory fields)]
doc_id / version / status / owner / last_updated / phase /
depends_on / supersedes / implementation_status / template_version

[12-item Self-check (from prompts/design-refine.md)]
1. Total line count significantly reduced (if optimization scenario).
2. All pseudocode blocks deleted or shortened to ≤ 10 lines.
3. §2 Component table divided into "Implemented / Planned."
4. §3 Component design uses decision tables instead of code blocks.
5. §6 Each risk has ✅/⚠️/🔜 markers and target slice.
6. §7 Verifiable criteria with [x] for completed and [ ] for planned.
7. §8 Includes four categories (Created / Planned New / Planned Mod / Not Created).
8. Chapter numbering is continuous with no skips or duplicates.
9. File paths match actual codebase (Verified via Grep before referencing).
10. All external project names/references marked as "Conceptual" not code porting.
11. No redundant/parallel data structures detached from runtime data model.
12. Current actual code status for target phase read and verified.

[Terminology Spec (MUST use term, synonyms allowed, NO other aliases)]
[Skill-populated from schema.glossary[], format:]
- Session (Synonyms: 会话, session object) — Persistent state container for a single session.
- Phase   (Synonyms: 阶段)               — Phase code within a version.
- ...

[Project Constraints (Inviolable boundaries)]
[Skill-populated from schema.project_constraints[] + inputs.constraints.session_only[], format:]
- [PC-001 / positioning] AwesomeApp is Enterprise Server (Core Engine + API Service), not IDE proxy.
- [PC-002 / reference_usage] Reference industry-standard/peer-system for concepts ONLY; do not port code.
- [PC-003 / scope_exclusion] No Plugin System replication.
- [SESSION-1 / scope_exclusion] Only align PTC, no sandbox this round.
Rework if any constraint is violated. All "Adoption Suggestions" must pass constraint filter.

[Adoption Criteria (from prompts/reference-extraction.md)]
- "Nice-to-have" capabilities from reference → Postpone, write to §0.6 Postponement Redlines.
- Core mechanisms from reference → Adopt, but prune to minimum viable for current project.
- Heavy infrastructure (if project lacks it) → Seek lightweight alternatives or postpone.

[Hard Constraints (Rework if violated, NO negotiation)]
1. No Creating Code: Do not write > 10 lines of pseudocode for explanation.
2. No Architecture Rewriting: Do not propose components/APIs/dependencies outside template.
3. No Deleting Motivation: Background, Problem statement, Cross-slice dependencies must stay.
4. No Exaggerating Status: Unverified features must NOT be marked [x].
5. No Mixing Fix Layers: File-level vs. Application-layer must be explained separately.
6. Preserve Postponement Redlines: "Postponement / NOT in Slice X" from original doc must stay.
7. No Losing Boundary Constraints: "Non-goals," perf/latency caps, module isolation redlines from original doc must be migrated to §0.6 or §7, not silently deleted.

====== Spec Injection Block End ======
```

#### §5.3.2 autopilot Command (Batch Optimizing Existing Docs)

```
/oh-my-claudecode:autopilot

Optimize following documents through multi-round autoresearch:
[Doc list]

Optimization Principles:
- Reference implementation of [Project A] [Project B]; gap analyze and combine with current best.
- Guard against over-engineering.
- Run the 12 self-check items in the "Spec Injection Block" after each round.
- Converge when 2 consecutive rounds pass all checks.
- If items still fail after N rounds (default 5), mark doc for manual handling and proceed.

====== Spec Injection Block (Skill Generated, DO NOT delete/edit) ======

[Same 6 sub-blocks as §5.3.1: Skeleton / Metadata / 12-item Check / Terminology / Adoption / Hard Constraints]

====== Spec Injection Block End ======
```

#### §5.3.3 Spec Block Reuse

Both commands share the same Spec Injection Block. The skill implementation **MUST** generate this from single sources (`templates/` + `prompts/` + `schema.glossary`) at runtime. Do NOT use hardcoded copies in the workflow, as they will become outdated when templates/prompts are upgraded.

### §5.4 Output Verification Checklist

After batch execution, verify each output document. **The first 2 items are Spec Injection Compliance; failure means immediate rework before further checks**:

- [ ] **Spec Skeleton Implemented**: §0-§8 all exist, in order, no duplicates (Cross-check with §5.3 chapter block).
- [ ] **Zero Violation of Hard Constraints**: Verify all 6 constraints (Pseudocode ≤ 10 lines, no external components, status not exaggerated...).
- [ ] Metadata header complete (per `templates/doc-metadata-schema.md`).
- [ ] Metadata contains `template_version` (Indicating omc adopted current skill spec).
- [ ] Passes 12 self-check items in `prompts/design-refine.md`.
- [ ] Chapter numbering continuous / no duplicates.
- [ ] Pseudocode blocks ≤ 10 lines (Mark for rework if exceeded).
- [ ] Consistent status markers (✅ / ⚠️ / 🔜 / 📋).
- [ ] References marked as "Conceptual Reference" (not code porting).
- [ ] File paths match codebase (Verified via Grep).
- [ ] Batch doc terminology aligns with `schema.glossary[]` (Unregistered term ratio > 10% → Rework and prompt user to update glossary).

### §5.5 Failure Recovery

| Failure Type | Handling |
|:---|:---|
| Single doc quality failed | Fall back to §2 Single-Doc Loop. |
| Batch convergence failed (Autopilot > N rounds) | Fall back to manual batching (≤ 3 docs per batch). |
| Multi-doc terminology conflict | Add Class C incremental item in `design-plan.md` §1.3; unify in next round. |
| Reference benchmarking deviation | Return to `prompts/reference-extraction.md` to re-extract key points. |

### §5.6 Output Write-back

After batch completion:
1. Update `design-plan.md` §4 Progress Tracking Table (Completion date for each doc).
2. Update `last_batch_run` timestamp in `{METADATA_DIR}/config.json`.
3. Trigger full sync check (`workflows/05-sync.md` Mode 3) to verify drifts.

---

## §6 Constraints

1. **No Skipping Self-checks**: Batch mode must also pass the `design-refine.md` checklist doc-by-doc.
2. **"AI Review Pass" ≠ "Human Finalized"**: `status` stays `in_review` until a human reviewer approves.
3. **No Unilateral Architecture Changes**: Batch optimization only performs "Align + Prune." If redesign is needed, escalate to §4 Multi-model Review.
4. **No Bypassing Spec Injection Block**: Calls to omc/omx MUST include the full §5.3.0 block. If user manually edits it, prompt again before calling and record to `session_changes[]` with `skill_override` tag.
5. **No Hardcoded Spec Block Copies**: Spec block content must be pulled at runtime from `templates/` / `prompts/` / `schema.glossary`. Examples in the workflow are for **format only**, not the authoritative source.
6. **No Losing Boundary Constraints**: If "non-goals," perf/latency caps, or module isolation redlines are found during optimization (any mode), they MUST be preserved in §0.6 or §7. **Do NOT silently delete them while pruning pseudocode** (per `design-refine.md §Hard Constraint #8`).
