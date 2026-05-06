<!--
  Purpose: Template for a Development Plan.
  Function: Tracks implementation tasks linked to design documents, organized by phases. Monitors progress and registers drifts.
  Last Modified: 2026-05-06 12:05:00 (CST)
-->
# [Project] [Version] Development Plan

> This plan mirrors the structure of `design-plan.md`; every development task must explicitly trace back to specific chapters of a design document.

| Field | Value |
|:---|:---|
| plan_id | `[project]-[version]-dev-plan` |
| version | `0.1` |
| owner | `[Development Lead]` |
| last_updated | `YYYY-MM-DD` |
| linked_design_plan | `[Path to corresponding design-plan.md]` |

---

## 1. Overall Overview

### 1.1 Milestones

| Milestone | Goal | Deadline | Status |
|:---|:---|:---|:---:|
| M1 | [Goal] | [Date] | not_started |

### 1.2 Task Status Summary

| Task ID | Title | Design Doc | Section | Priority | Status | Owner |
|:---|:---|:---|:---|:---:|:---:|:---|
| T-001 | [Task] | `[doc-id]` | §3.1 | High | not_started | [Name] |

Status enum: `not_started`, `in_progress`, `blocked`, `in_review`, `done`.

---

## 2. Task Batches and Dependencies

### 2.1 Dependency Graph

```
T-001 ──▶ T-003
         ▲
T-002 ───┘
```

### 2.2 Batch Scheduling

#### Batch 1 — [Phase Name]

| Task ID | Title | Dependencies | Est. Effort |
|:---|:---|:---|:---:|
| T-001 | [Task] | — | 2d |

#### Batch 2 — ...
<!-- Same as above -->

---

## 3. Task Specifications

### 3.1 T-001: [Task Title]

**Design Reference**: `[doc-id]` §3.1 — §3.2

**Deliverables**:
- Code paths: `crates/.../xxx.rs`
- Tests: `cargo test xxx --lib`
- Doc updates: If implementation deviates from design, sync update design doc §X.Y.

**Acceptance Criteria** (from design doc §7):
- [ ] [Criterion 1]
- [ ] [Criterion 2]

**Implementation Notes** (Append on completion):
- [Date] [Brief record]

### 3.2 T-002: ...
<!-- Same as above -->

---

## 4. Progress Tracking

### YYYY-MM-DD

- [done] T-001 finished; submitted commit `abc1234`.
- [blocked] T-003 waiting for interface stability of T-001.
- [drift] Discovered discrepancy between `[doc-id]` §3.1 and implementation; fed back to design-plan.

---

## 5. Collaboration with Stage 4 Sync

Completion of tasks in this plan triggers built-in verification (`workflows/05-sync.md` Mode 1) to check for drifts:

- Every `done` task → Run incremental sync (scanning only files involved in T-XXX).
- Handling generated drifts:
  - **Code Leading Doc**: Supplement description in design doc, or rollback code.
  - **Doc Error**: Register Category A issue in `design-plan` and schedule revision.

---

## 6. Drift Registry

Mirror view of drifts imported from `{METADATA_DIR}/state.json.drifts` (Read-only):

| drift-id | Doc | Code Path | Severity | Strategy | Task Link |
|:---|:---|:---|:---:|:---|:---|
| drift-001 | `[doc-id]` §3.1 | `xxx.rs:42` | P0 | Fix Doc / Fix Code | T-005 |
