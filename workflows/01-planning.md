<!--
  Purpose: Workflow for Stage 1 — Goal-Driven Planning (Design & Dev Plans).
  Function: Guides the user in creating systematic optimization plans for multiple documents or development task lists, ensuring goal alignment and dependency awareness.
  Last Modified: 2026-05-06 10:45:00 (CST)
-->
# Stage 1 — Planning (Design Plan / Development Plan)

> Purpose: Generate traceable document system optimization plans or development task plans for complex projects.

## When to Use

| Mode | Scenario | Output Artifact |
|:---|:---|:---|
| **Mode A: New Version Planning** | Starting a new version or major evolution involving ≥ 5 design documents. | `design-plan.md` |
| **Mode B: Implementation Handoff** | Design documents are finalized; entering implementation phase and need dev task management. | `dev-plan.md` |
| **Mode C: Goal-Driven Gap Analysis** | User provides a clear goal (e.g., "Align implementation with industry-standard"), needing a derived document list. | `design-plan.md` (incremental) |
| **Mode D: Systematic Revision** | Multiple existing documents have drifted from the codebase. | `design-plan.md` (revision-focused) |

Revisions for a single document do not need Stage 1; go directly to Stage 2.2 (Optimization).

## Prerequisites

**`{METADATA_DIR}/project-schema.json` must exist and `document_inventory[]` must not be empty.** Otherwise, redirect to `workflows/0 adoption.md`.

If `roadmap_references[].treated_as="design_plan"` exists in the schema, it's identified as an existing roadmap. This stage will prioritize the "Optimize Existing Roadmap" branch (see §New Version / Existing Roadmap Branch) instead of generating from a blank template.

---

## Inputs

Prioritize reading from `{METADATA_DIR}/project-schema.json`:
- `project_name` / `scope.active_versions` → Determine target version.
- `document_inventory[]` → Metadata of known docs (status / code_paths / phase).
- `roadmap_references[]` → Existing roadmaps to be used as design plans.
- `config.references` → Reference sources added in Stage 0 or previous rounds.

Supplement via `AskUserQuestion` based on the schema:

1. **Phase Code for this round** (e.g., "V10 Phase A").
2. **Baseline Documents** (if any): `doc_id` of documents acting as "anchors" that should not be modified.
3. **New Reference Sources** (incrementally written back to config).
4. **Current Implementation Overview**:
   - Priority/rhythm constraints for the current phase.
   - Known quality issue categories (A/B/C/D).

**Do NOT repeat** information already in the schema to avoid redundant questions.

---

## Process Steps

### Step 0 — Identify Work Goal (Mandatory Pre-condition)

**A goal must be confirmed before any mode begins.** Detect the current session context:

| Signal | Inference |
|:---|:---|
| User message contains clear goal ("Align gap for...", "Implement X capability", "Align with Y project") | Goal is present; proceed to Step 0.1 Extraction. |
| User only says "Create design plan" / "Planning" without a specific goal | **Must** use `AskUserQuestion` to elicit goal. |
| `current_goal` field exists in `schema.scope` | Reuse and ask user for confirmation. |

#### Step 0.1 — Extract & Solidify Goal

Structure the goal as:

```
goal_id: G-<short-slug>
title: <One-line description>
type: feature_alignment | gap_closure | new_capability | refactor | sync_fix
reference_baseline: <Benchmark project/code/doc>, optional
expected_outcome: <Verifiable output, e.g., "V11 alignment with industry-standard skill loading mechanism">
constraint: <Boundaries for this goal, e.g., "Do not modify V10 released parts">
```

Write to `{METADATA_DIR}/state.json.current_goal` for reuse in later Stages.

**Simultaneously verify `schema.project_constraints[]`**:
- If `project_constraints[]` is missing or empty in the schema, and the goal's `reference_baseline` includes external projects (e.g., industry-standard / peer-system), **must AskUserQuestion** to guide the user in registering at least 3 types of project-level constraints:
  - `positioning` (Project boundaries vs. reference project).
  - `reference_usage` (How to use references: Concept vs. Code Porting vs. Independent).
  - `scope_exclusion` (Explicitly excluded capabilities, e.g., "No IDE-side system replication").
- Append answers to `schema.project_constraints[]` with stable `PC-NNN` IDs.
- If `current_goal.constraint` conflicts with project constraints, correct the goal first rather than adding a new project constraint.

#### Step 0.2 — Guidance for Missing Goals

If the user hasn't provided a goal, ask in the following order (one at a time to avoid fatigue):

1. "What is the core problem we are solving this round? (e.g., Adding X capability / Aligning with Y project / Fixing Z drifts)"
2. "Are there benchmark references? Local code paths or external projects?"
3. "What are the verifiable success criteria?"
4. "Are there boundaries that cannot be touched (Released versions / Locked APIs)?"

Confirm collected info with the user before entering the corresponding Mode.

#### Step 0.3 — Automatic Mode Determination

Based on goal `type` and schema state:

```
type=new_capability + scope.active is empty     → Mode A
type=feature_alignment | gap_closure            → Mode C (Default)
type=sync_fix                                   → Mode D
dev-plan missing but design is approved         → Mode B
```

Report the determination to the user and allow override.

---

### Step 0.5 — Mode C Exclusive: Goal-Driven Gap Analysis

> Execute only for Mode C; others skip to Step 1.

1. **Load Baseline**:
   - `Read` the `reference_baseline` declared in the goal (e.g., industry-standard local path).
   - Ensure the path is registered in `{METADATA_DIR}/config.json.references`; otherwise, append it.
2. **Inventory Current Status**:
   - Read all `active` version docs + `code_paths` from `schema.document_inventory[]`.
   - `Grep` the current repo to verify `code_paths` actually exist.
3. **Inventory Benchmark Capabilities**:
   - Use `prompts/reference-extraction.md` to extract capability lists from the reference project based on the goal.
4. **Generate Gap Matrix**:

   | Capability | Our Doc | Our Code | Benchmark Impl | Gap Type |
   |:---|:---|:---|:---|:---|
   | Capability X | doc-id | crates/foo | refs/foo.rs | Missing Doc / Missing Code / Alignment Drift |

5. **Decompose Goal into Plan Items**: Every gap must correspond to at least one `design-plan.md` item (New doc / Revised section / New dev task), written to §1.3 Issue Classification.

**Constraints**:
- The "Our Status" column in the gap matrix must be based on `Grep/Read` evidence, not just schema inference.
- Benchmark capabilities must list source line numbers for auditability.
- Mode C defaults to outputting **incremental items** for `design-plan.md` (append to existing plan), not overwriting existing content.

---

### Step 1 — Identify Current Status

1. Retrieve all registered documents (filtered by `scope`) from `{METADATA_DIR}/project-schema.json.document_inventory[]`.
2. For each document:
   - Read existing metadata (status / implementation_status / score) from the inventory.
   - Open the file and read the section outline.
   - Verify the reality of `inventory.code_paths`.
3. If `{METADATA_DIR}/state.json` exists, read the `drifts` field for historical discrepancies.

---

### Step 2 — Generate Scoring Table

Score each document across four dimensions (Accuracy / Completeness / Actionability / Consistency), 0-10 each.

**Scoring Criteria**:
- **Accuracy**: Code path existence rate, implementation status accuracy.
- **Completeness**: Coverage of components, risks, acceptance, and file planning.
- **Actionability**: Pseudocode density (lower is better), decision table density (higher is better).
- **Consistency**: Alignment with baseline doc positioning and terminology across concurrent docs.

Output: Fill the summary table in `templates/design-plan.md` §1.2.

---

### Step 3 — Issue Classification

Categorize low-score items into A/B/C/D classes (see Template §1.3):
- **Class A (Critical Fix)**: Conflicts with implementation; will mislead future development.
- **Class B (Key Missing)**: Missing sections preventing implementation.
- **Class C (Partial Revision)**: Terminology / Positioning misalignment.
- **Class D (Polishing)**: Enhancements that do not block progress.

---

### Step 4 — Dependency Layering

Divide documents into Layers 0-5 based on "change propagation impact." **Method**:
- Authority: `project-schema.json.document_inventory[].depends_on`. If missing, **must** complete first (ask user or infer from `depends on / based on` keywords in doc text and confirm with user).
- Perform BFS traversal of the `depends_on` directed graph starting from baseline docs.
- Docs referenced by many are placed in lower Layers.
- Docs referencing others but not being referenced are placed in higher Layers.
- If a cycle is found → Abort writing immediately and ask for user decision to break the cycle.
- Perform **glossary consistency check**: `Grep` all affected docs for `schema.glossary[].term` and `synonyms`. Prompt user to append high-frequency unregistered terms. Mark docs revising existing terms as Class C issues.

---

### Step 5 — Batch Scheduling

Schedule in ascending order of Layers, 3-5 docs per batch. Workload estimation:
- **Small** (< 0.5 day): Adding sections / correcting terminology.
- **Medium** (0.5-1 day): Completing full chapters.
- **Large** (1-2 days): Rewriting core chapters or multi-chapter coordination.

---

### Step 6 — Update Config & Schema

**config** Rules (Incremental if adoption already created):
- Append new `references.local_repos[]` / `references.online_docs[]` for this round.
- Adjust `sync.check_types_default` if necessary.

**schema** Sync (Automatic):
- If the plan introduces new versions → bump `scope.active_versions`.
- If docs missing from inventory are found → prompt to refresh adoption (do not write in this step).
- Update `last_synced_at`.

**Do NOT repeat** `project_name` / `scope` / `design_doc_roots` — the schema is the source of truth for these.

---

### Step 7 — Output Plan File (Idempotent Write)

Target path: `[Doc Root]/design-plan.md` or `dev-plan.md`.

| File Status | Behavior |
|:---|:---|
| Not Found | Create using `templates/*.md` and fill results. |
| Exists + Mode A (New Version) | Ask user: Archive old file to `*-plan.archived-<date>.md` before creating new; **Do NOT overwrite silently**. |
| Exists + Mode C / D (Incremental) | `Edit` to append new items to sections (§1.3 Issues, §2 Schedule), preserving existing structure. |
| Exists + Mode B (dev-plan) | Append new task items only; do not touch status/notes of existing tasks. |

**Prohibit** whole-file overwriting of existing plans. Every addition/revision must be recorded in `state.json.session_changes[]` for rollback.

---

## Step 8 — Next Steps Prompt

```
✅ Plan Generated: [Path]
config Updated: {METADATA_DIR}/config.json
schema last_synced_at Updated

Recommended Next Steps:
  - Batch 1, Doc 1: [doc-id] (from schema.document_inventory)
  - Switch to Stage 2: workflows/02-design-authoring.md
  - Reference sources auto-injected from config.references
  - If existing docs have never been synced, run a full check first (workflows/05-sync.md Mode 3) to detect existing drifts.
```

---

## Constraints (Mandatory)

1. **No Blind Scoring**: Every score must be based on actually reading doc content.
2. **No Skipping Metadata Check**: Docs without metadata must be marked and prioritized in the plan for completion.
3. **No Rewriting Docs in Plan**: Stage 1 only outputs the plan; modifications happen in Stage 2.
4. **Explicit References**: Do not use "experience" alone as a basis for decisions.
5. **No Skipping Goal ID**: Do not proceed past Step 0 without confirming `current_goal`.
6. **No Silent Overwrite**: Follow Step 7 idempotent write rules.

---

## Development Plan (dev-plan.md) Differences

Additional steps for dev-plan generation:

1. **Task Decomposition**: Every deliverable chapter in the design doc → at least one dev task.
2. **Back-linking**: Every task must be tagged with corresponding `doc_id + section`.
3. **Drift Registration**: Import known drifts from `{METADATA_DIR}/state.json.drifts` as a read-only view (Template §6).
4. **Acceptance Binding**: Task acceptance criteria must directly reference design doc §7, do not rewrite.
