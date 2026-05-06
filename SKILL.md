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

## Startup Detection (Mandatory First Check)

**Before serving any user request**, check `{METADATA_DIR}/project-schema.json`:
- If file does not exist: **Must** run `workflows/00-adoption.md` first.
- If file exists but `document_inventory` is empty: Treated as first-time (run adoption).
- If metadata partially missing: Run `workflows/00-adoption.md §Path R` (refresh gaps only).
- If goal is unclear: Trigger `workflows/01-planning.md §Step 0.2` goal elicitation.
- If goal is clear: Proceed to requested stage.

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
| **2. Design** | `02-design-authoring.md` + `03-design-optimization.md` | Accurate, optimized design docs per template |
| **3. Develop** | `workflows/04-dev-execution.md` | Code implement traceable to design, with dev-plan progress |
| **4. Sync** | `workflows/05-sync.md` | Built-in drift detection driven by project-schema |

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
10. **Review independence, with graceful degradation**: Multi-model review (L1-L3) per `prompts/multi-model-review.md`.
11. **Revision-based writes**: Every write to JSON metadata is gated by an optimistic `revision` counter (CAS).
12. **Reversible by default**: Every mutating step appends to `state.json.session_changes[]` with `undo_hint`.
13. **External orchestrator compliance via prompt injection**: When delegating to `omc/omx`, inject conventions and constraints inline into the prompt.
14. **Constraints-as-norm**: `schema.project_constraints[]` are inviolable boundaries. Every suggestion must pass the constraint filter.
15. **Depth-by-default for quality optimization**: Defaults to the 9-step deep path in `03-design-optimization.md` for quality requests.

## Data Protocols

### Concurrency (optimistic CAS via `revision`)

`project-schema.json` and `state.json` carry a top-level `revision`. Writes follow:
1. Read current `revision` (r0).
2. Compute new content.
3. Re-read revision; if still r0, write with `revision = r0 + 1`.
4. If mismatch after 3 attempts, surface conflict to user.

### Rollback (structured `session_changes[]`)

Mutating steps append to `session_changes[]`. `undo_hint.kind` is one of `text_revert` / `json_patch_revert` / `file_created` / `file_renamed`. `workflows/99-rollback.md` replays these in reverse; reverted entries are marked `reverted: true` (never deleted).

### Authoring Checkpoint (resume long sessions)

`state.json.authoring_checkpoints[doc_id]` tracks in-flight authoring sessions to allow resuming from the last completed section. Cleared upon approval.

### Template & Prompt Versioning

Tracks `template_version` and `prompt_version` in document metadata. Stage 4 sync reports `template_out_of_date` warnings if a doc was created with an older skill version.

### Terminology (Glossary)

`project-schema.json.glossary[]` centralizes canonical terms. Docs must not redefine existing terms. `pattern_check` in Stage 4 detects term drift across the codebase.

### Document Dependencies (`depends_on`)

Autoritative `depends_on[]` in inventory. Stage 1 uses this for layering; Stage 4 warns on dependency inversion (child `approved` while parent is `draft`). Circular dependencies are rejected.

---

## Configuration & State

- **`project-schema.json`**: Project fingerprint (Conventions, scope, inventory).
- **`config.json`**: Skill behavior (Host, sync rules, references).
- **`state.json`**: Dynamic state (Current phase, tasks, drifts, session log).

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
- [ ] Stage 2 design docs pass 12-item self-check and respect glossary terms.
- [ ] Stage 3 Step 1.5 code-reality probe completed; drifts logged.
- [ ] Stage 3 dev-plan tasks link back to specific doc_id + section.
- [ ] Stage 4 drift report is fresh.
- [ ] `schema.project_constraints[]` registered (≥1 positioning, ≥1 reference_usage).
- [ ] "Optimization" requests followed the deep path by default.
- [ ] Implementation sections in deep/batch-deep path produce "Implementation" semantic chapters per `03-design-optimization.md §1.6` morphological constraints (decision tables / module maps / interface contracts ≤10 lines / data flows / error matrices / edge case checklists); NO pseudocode blocks >10 lines, NO function bodies, NO crates/types/paths that don't exist in the project; chapter titles do NOT include language names (e.g., "Rust Implementation").
