<!--
  Purpose: Definition of the doc-driven-dev skill.
  Function: Defines core responsibilities, interaction primitives, data protocols, and constitutional principles.
  Last Modified: 2026-05-06 12:20:00 (CST)
-->
---
name: doc-driven-dev
version: 2026.5.6
description: Closed-loop document-driven development orchestrator. Adopts greenfield or brownfield projects via a project schema (no intrusion on existing user docs), then guides design planning, authoring, multi-model optimization, reference-based development execution, and ongoing doc-code sync. Use whenever creating, refining, or executing against technical design documents (PRDs, RFCs, architecture specs, phased delivery plans), especially when projects involve ≥ 5 related docs, cross-project reference takeaways, multi-model review cycles, or need traceable progress from design to code. Triggers: "设计文档", "design doc", "演进设计", "RFC", "制定设计计划", "批量优化设计文档", "按设计文档开发", "多模型评审", "文档驱动开发", "在这个项目启用 doc-driven-dev", "接管项目文档体系", "adopt existing project docs".
when_to_use: Design plan creation, design doc authoring, design doc multi-model optimization, dev execution with plan tracking, ongoing doc-code sync verification, reference-project-driven takeaways
argument-hint: "[plan|design|dev|sync] [--stage N] [--reference path-or-url]"
allowed-tools: Read, Write, Edit, Grep, Glob, Bash, AskUserQuestion
---

# doc-driven-dev Skill

Document-driven development closed-loop orchestrator. Ensures every line of code traces back to an accurate, optimized design document, and every design document traces back to real project context and reference-project takeaways.

---

## ⚡ Session Pre-flight (EXECUTE FIRST — NO EXCEPTIONS)

**STOP. Before reading anything else in this skill or doing any work, execute the following Read calls in order:**

```
Step PF-1: Read  {METADATA_DIR}/config.json
Step PF-2: Read  {METADATA_DIR}/project-schema.json
Step PF-3: Read  {METADATA_DIR}/state.json
```

Then read the plan docs registered in `config.json.plan_docs`:

```
Step PF-4: Read  config.plan_docs.design_plan   (e.g. Docs/He-agentDoc/design-plan.md)
Step PF-5: Read  config.plan_docs.dev_plan       (e.g. Docs/He-agentDoc/dev-plan.md)
```

After completing all five reads, **output this exact confirmation block** (fill in real values from the files):

```
[doc-driven-dev Pre-flight]
models       : <config.json .models[]>
peer_projects: <for each entry, try each path in .paths[] in order, report first existing path; mark MISSING if none found>
constraints  : <count> entries loaded (pc-001…)
principles   : <count> entries loaded (pp-001…)
phase        : <state.json .current_phase>
goal         : <state.json .current_goal.title  (or .current_goal if stored as string)>
design_plan  : <config.plan_docs.design_plan path, confirm file read>
dev_plan     : <config.plan_docs.dev_plan path, confirm file read>
```

**Do NOT proceed to the user's actual request until this block is printed.**

If any peer project path resolves to MISSING, warn the user immediately:
> ⚠️ Peer project `<role>` not found at any configured path. Reference comparison will be incomplete.

If `models` is empty, stop and ask the user for preferred model(s) before continuing.

Then execute **Step PF-6: Startup Routing Decision** based on the loaded data:

```
IF project-schema.json does not exist or document_inventory is empty:
    → Run Stage 0 Adoption (inline, see §Adoption Required Outputs)
ELSE IF project_constraints[] is missing or empty:
    → Flag constraints_gap; run Adoption §Constraints refresh before proceeding
ELSE IF state.json.current_goal is empty or unclear:
    → Ask user: "What is the goal for this session?" and persist to state.json.current_goal
ELSE:
    → Proceed to the stage matching the user's request
```

---

## When to Use

Use this skill when:
- Starting a complex project or major version where multiple related design documents need planning.
- Writing a high-quality design document from scratch with reference projects as context.
- Existing design documents drift from implementation and need correction + optimization.
- Executing development against a design document and tracking progress.
- Running multi-model review loops to converge on an optimal design.

**Do NOT use** for:
- Simple code tasks without design artifacts (use regular coding skills instead).
- Pure documentation without implementation intent (use general writing).

## Metadata Directory Resolution (Host-aware)

The skill writes its own metadata under a host-specific directory, resolved **once per session** and cached in `config.json`. Referred to as `{METADATA_DIR}` throughout this skill.

### Resolution order

| Condition | `{METADATA_DIR}` |
|:---|:---|
| `.claude/` directory exists at project root (Claude Code) | `.claude/doc-driven-dev/` |
| `.codex/` directory exists at project root (Codex) | `.codex/doc-driven-dev/` |
| Both exist | Ask user once, persist choice to `config.json.host` |
| Neither exists (Windsurf, Cursor, plain IDE, etc.) | `.doc-driven-dev/` |

The resolved directory contains three files:
- `{METADATA_DIR}/project-schema.json` (Project fingerprint)
- `{METADATA_DIR}/config.json` (Skill behavior)
- `{METADATA_DIR}/state.json` (Dynamic state)

### External orchestrator boundaries (DO NOT touch)

The skill can **call** `oh-my-claudecode` / `oh-my-codex` as external batch orchestrators. These tools maintain their **own private metadata** (e.g., `.omc/`, `.omx/`).

**Strict rule**: The skill must **never read or write `.omc/` or `.omx/` directly**. These directories are owned by the external tools; the skill only invokes them and consumes their output.

## Five-Stage Closed Loop

```
Adoption ---▶ Plan ---▶ Design ---▶ Develop ---▶ Sync ---┐
   ▲                                                    │
   └────────────── schema refresh on scope change ─┘
                                                       │
                  drift feedback revises plan ◀──────┘
```

| Stage | Workflow File | Output |
|:---|:---|:---|
| **0. Adoption** | `workflows/00-adoption.md` | `{METADATA_DIR}/project-schema.json` |
| **1. Plan** | `workflows/01-planning.md` | `design-plan.md` / `dev-plan.md` with scoring, graph, batch order |
| **2. Design** | `workflows/02-design-authoring.md` + `workflows/03-design-optimization.md` | Accurate, optimized design docs per template |
| **3. Develop** | `workflows/04-dev-execution.md` | Code implement traceable to design, with dev-plan progress |
| **4. Sync** | `workflows/05-sync.md` | Built-in drift detection driven by project-schema |

### Stage 2 Mandatory Project Context Loading

**Before any authoring or optimization step**, load and apply:

1. `project-schema.json.project_constraints[]` → inject into `inputs.constraints.from_project` (NOT `session_only`); every output suggestion must pass these constraints.
2. `project-schema.json.project_principles[]` → apply as a pre-output filter checklist; flag violations before writing.
3. `config.json.references.peer_projects[]` → merge with any session-provided `reference_projects`; **all entries are mandatory regardless of `role`**. Each entry uses a `paths[]` array for cross-platform support: resolve by trying each path in order and using the **first one that exists** on the current OS. For each resolved peer project, read the corresponding implementation, then compare side-by-side: take the stronger pattern from each (取长补短). `role: primary_reference` gets higher alignment weight; `role: secondary_reference` must still be consulted and its relevant parts explicitly cited. Never skip a peer project because its role is "secondary". Goal: arrive at the solution optimal for the current project's architecture, not a copy of any single reference.
4. `config.json.models[]` → use as the default model list for multi-model review; supplement with session overrides only.

### Post-change Sync Protocol

**After completing any design doc change, execute ALL applicable sync steps before closing the session. Partial sync creates split-brain state and is not acceptable.**

#### When a design doc is optimized or its status changes:

| Target | Fields to update |
|:---|:---|
| `project-schema.json` `.document_inventory[doc_id]` | `score`, `status`, `last_updated` |
| `state.json` `.session_changes[]` | Append entry: `stage`, `action`, `target`, `summary` (CAS-guard `revision`) |
| `design-plan.md` §1.2 row for this doc | `quality.total` or `deep_review.total`, `当前状态`, `最近评分`, `备注` |
| `design-plan.md` §4 Progress Tracking row | status 和 completion date |
| `design-plan.md` §5 Execution Log | Append `[YYYY-MM-DD] {doc-id} {mode} optimization done, score {old}→{new}` |
| `dev-plan.md` §1.2 row for this doc | `设计` column — update to `done` when status=approved, `in_review` otherwise |

#### When a dev task progresses:

| Target | Fields to update |
|:---|:---|
| `project-schema.json` `.document_inventory[doc_id]` | `implementation_status`, `code_paths[]`, `last_updated` |
| `state.json` `.session_changes[]` | Append entry (CAS-guard `revision`) |
| `dev-plan.md` §1.2 row for this task | `开发` column, `状态` |

Plan doc paths are in `config.json.plan_docs`. Read the relevant section before editing—never overwrite the whole file. Use targeted Edit calls on the specific table row.

### Adoption Required Outputs

The adoption workflow (`workflows/00-adoption.md`) **must** collect and persist all of the following before closing:

| Field | Location | Ask-if-missing prompt |
|:---|:---|:---|
| `models[]` | `config.json` | "Which LLM(s) for multi-model review? (e.g., claude-sonnet-4-6)" |
| `references.peer_projects[]` | `config.json` | "Are there reference projects? Provide paths for each OS (Windows + macOS) and their role." |
| `project_constraints[]` | `project-schema.json` | "List hard architectural/technology constraints (≥1 positioning, ≥1 technology)." |
| `project_principles[]` | `project-schema.json` | "List project-wide design principles for all docs (e.g., doc structure rules, forbidden patterns)." |

## Core Principles

1. **Schema-driven, not doc-intrusive**: Reads project-schema; never requires user docs to change format.
2. **Reference-guided**: Persists reference sources (projects, URLs) for reuse across stages.
3. **Status-layered**: Components / risks / acceptance criteria must distinguish "Implemented / Partial / Planned."
4. **Anti-over-engineering**: Benchmark reference best-practices but prune to current project scope.
5. **Progressive disclosure**: Greenfield uses embedded headers; Brownfield uses external inventory.
6. **Zero-intrusion brownfield**: Discovery is read-only unless the user opts in to modifications.
7. **Idempotent-by-default**: Every stage fills gaps and appends increments; never overwrites existing work.
8. **Goal-first**: Stage 1 must elicit and persist `current_goal` in `state.json`.
9. **Code-reality verified before coding**: Stage 3 requires a code probe (`Step 1.5`) before implementation.
10. **Review independence, with graceful degradation**: Multi-model review (L1/L2/L3) per `prompts/multi-model-review.md §0.2`. Graceful degradation: fewer available models → lower level; single model → multi-persona mode per `§0.4`.
11. **Revision-based writes**: Every write to JSON metadata is gated by an optimistic `revision` counter (CAS).
12. **Reversible by default**: Every mutating step appends to `state.json.session_changes[]` with `undo_hint`.
13. **External orchestrator compliance via prompt injection**: When delegating to `omc/omx`, inject conventions and constraints inline into the prompt.
14. **Constraints-as-norm**: `schema.project_constraints[]` are inviolable boundaries. Every suggestion must pass the constraint filter.
15. **Depth-by-default for quality optimization**: Defaults to the 9-step deep path in `03-design-optimization.md` for quality requests.
16. **Clean-output-only**: Design document files must contain only clean, final content. Strikethroughs (`~~...~~`), editorial annotations (已删除 / 已修复 / 已更新 / \[FIXED\] / \[DELETED\] / `<!-- review comment -->`), diff markers, and any review-process residue must **never** appear in the written design doc. All change tracking belongs exclusively in `state.json.session_changes[]` and `review_session` entries. When rewriting a section, output the corrected content directly—do not show before/after diffs inline.

## Data Protocols

### Concurrency (optimistic CAS via `revision`)

`project-schema.json` and `state.json` carry a top-level `revision`. Writes follow:
1. Read current `revision` (r0).
2. Compute new content.
3. Re-read revision; if still r0, write with `revision = r0 + 1`.
4. If mismatch after 3 attempts, surface conflict to user.

### Rollback (structured `session_changes[]`)

Mutating steps append to `session_changes[]`. `undo_hint.kind` is one of `text_revert` / `json_patch_revert` / `file_created` / `file_renamed`. `workflows/99-rollback.md` replays these in reverse; reverted entries are marked `reverted: true` (never deleted).

### Template & Prompt Versioning

Tracks `template_version` (SKILL.md version when doc was authored) in document metadata. Stage 4 Sync checks for `template_out_of_date` warnings by comparing doc's `template_version` against current SKILL.md `version` field. *(Requires `workflows/05-sync.md` or inline sync logic.)*

### Terminology (Glossary)

`project-schema.json.glossary[]` centralizes canonical terms. Docs must not redefine existing terms. Stage 4 Sync performs term-drift check: Grep all design docs for glossary terms, flag any redefinition.

### Review Session Schema (`review_session`)

`state.json.review_session` is a map keyed by `doc_id`. Each entry:
```
{
  "inputs": {
    "goal_kind": ["Fix accuracy"|"Complete structure"|"Align with reference"],
    "reference_projects": ["<resolved path>"],
    "benchmark_docs": ["<doc_id>"],
    "required_sections": ["§N title"],
    "constraints": { "from_project": ["<pc-id>"], "session_only": ["<text>"] },
    "review_level": "L1"|"L2"|"L3"
  },
  "level": "L1"|"L2"|"L3",
  "models_used": ["<model_id>"],
  "rounds": <int>,
  "converged": <bool>,
  "known_issues": ["<ERR-id> <description>"],
  "score": <float 0-10>,
  "status": "in_progress"|"audit_complete"|"pending_human_review",
  "adoption_suggestions": ["<text>"]
}
```
Write a new entry when starting a review session; update in-place until `status` reaches `pending_human_review`.

### Authoring Checkpoints

`state.json.authoring_checkpoints[doc_id]` stores:
```
{
  "sections_done": ["§0", "§1", ...],
  "sections_pending": ["§2", "§3", ...],
  "current_section": "§2",
  "last_updated": "<iso>",
  "notes": "<what to do next time>"
}
```
Updated after each section completes (CAS-guard). Cleared (`null`) when document reaches `status=approved`. Full usage protocol in `workflows/02-design-authoring.md §Step 2`.

### Document Dependencies (`depends_on`)

Autoritative `depends_on[]` in inventory. Stage 1 uses this for layering; Stage 4 warns on dependency inversion (child `approved` while parent is `draft`). Circular dependencies are rejected.

### Project Constraints Schema (`project_constraints[]`)

Each entry: `{id: "pc-NNN", kind: "positioning|technology|architecture|reference_usage", description: "...", inviolable: true|false}`.

- Adoption must register ≥1 `positioning` entry and ≥1 `technology` entry.
- Stage 2 loads this array before every optimization and validates all output against each entry.
- To relax a constraint, set `inviolable: false` (never delete entries).

### Project Principles Schema (`project_principles[]`)

Each entry: `{id: "pp-NNN", description: "..."}`.

- Capture project-specific doc authoring rules, naming conventions, and prohibited patterns.
- Applied as an output filter in Stage 2 authoring and optimization checks.
- Distinct from SKILL core principles (those are skill-level; these are project-level and project-specific).

---

## Configuration & State

- **`project-schema.json`**: Project fingerprint (doc conventions, scope, inventory, `project_constraints[]`, `project_principles[]`, `glossary[]`).
- **`config.json`**: Skill behavior (host, `models[]`, `references.peer_projects[]`, `plan_docs`, sync rules).
  - `plan_docs.design_plan`: path to design scoring dashboard (e.g. `Docs/He-agentDoc/design-plan.md`).
  - `plan_docs.dev_plan`: path to dev task tracking dashboard (e.g. `Docs/He-agentDoc/dev-plan.md`).
- **`state.json`**: Dynamic state (current phase, goal, `session_changes[]`, `review_session{}`, `authoring_checkpoints{}`, drifts).

## Quick Entry Points

| User Intent | Workflow |
|:---|:---|
| "Adopt existing project docs" / First-time setup | `workflows/00-adoption.md` |
| "Create design plan" / Systemic optimization | `workflows/01-planning.md` |
| "Write new design doc" / Section-by-section drafting | `workflows/02-design-authoring.md` |
| "Optimize design doc" / Batch optimization | `workflows/03-design-optimization.md` |
| "Develop against design doc" / Track progress | `workflows/04-dev-execution.md` |
| "Check consistency" / View drifts / Check progress | `workflows/05-sync.md` |
| "Multi-model review process" | `prompts/multi-model-review.md` |
| "Extract implementation from reference" | `prompts/reference-extraction.md` |
| "Undo last change" / Rollback | `workflows/99-rollback.md` |

---

## Operational Checklist

- [ ] `{METADATA_DIR}/project-schema.json` exists and is non-empty.
- [ ] Idempotent runs followed Path R (no overwriting existing work).
- [ ] `state.json.current_goal` recorded for planning/optimization stages.
- [ ] All metadata writes were CAS-guarded (`revision` bumped).
- [ ] Every mutating action produced a `session_changes[]` entry.
- [ ] Stage 1 output includes 4-dimensional scoring and `depends_on` layering.
- [ ] Stage 2 design docs pass 12-point audit from `prompts/design-refine.md` (run in `workflows/02-design-authoring.md §Step 3`), all project_principles[] entries verified, glossary terms respected.
- [ ] Stage 3 Step 1.5 code-reality probe completed; drifts logged.
- [ ] Stage 3 dev-plan tasks link back to specific doc_id + section.
- [ ] Stage 4 drift report is fresh.
- [ ] `schema.project_constraints[]` registered (≥1 positioning, ≥1 reference_usage).
- [ ] "Optimization" requests followed the deep path by default.
- [ ] Implementation sections in deep/batch-deep path produce "Implementation" semantic chapters per `03-design-optimization.md §1.6` morphological constraints (decision tables / module maps / interface contracts ≤10 lines / data flows / error matrices / edge case checklists; algorithmic pseudocode ≤20 lines with `<!-- pseudocode: language-agnostic, not for direct implementation -->` annotation as narrow exception); NO code blocks >20 lines, NO function bodies, NO crates/types/paths that don't exist in the project; chapter titles do NOT include language names (e.g., "Rust Implementation").
- [ ] `config.json.models` populated (≥1 model ID); deep optimization used this list, not an ad-hoc choice.
- [ ] `project-schema.json.project_constraints[]` populated (≥1 `positioning`, ≥1 `technology`); every optimization output validated against it.
- [ ] `project-schema.json.project_principles[]` populated (≥1 entry); applied as output filter during authoring and optimization.
- [ ] `config.json.references.peer_projects[]` populated if user ever mentioned reference projects; merged into every optimization session's `inputs.reference_projects`.
- [ ] Stage 2 optimization loaded project context (constraints + principles + references + models) before generating output; NOT relying on ad-hoc session inputs alone.
- [ ] Post-change sync completed for design doc optimization: `project-schema.json` inventory row updated + `design-plan.md` §1.2 row updated (score, status, 备注) + `dev-plan.md` §1.2 设计 column updated.
- [ ] Post-change sync completed for dev progress: `project-schema.json` implementation_status updated + `dev-plan.md` §1.2 开发 column + 状态 updated.
- [ ] No split-brain state: `project-schema.json` score/status matches `design-plan.md` §1.2 for the same doc_id.
- [ ] No strikethroughs, editorial annotations (已删除 / 已修复 / ~~...~~ / \[FIXED\] / \[DELETED\] / `<!-- review comment -->`), diff markers, or review-process markers left in any design document file; all change tracking recorded in `state.json` only.
- [ ] Design document files contain only clean, final content; all change tracking belongs exclusively in `state.json.session_changes[]` and `review_session` entries.
