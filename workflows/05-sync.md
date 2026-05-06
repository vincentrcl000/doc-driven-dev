<!--
  Purpose: Workflow for Stage 4 — Doc-Code Consistency Sync (Built-in).
  Function: Periodically or on-demand verifies that design documents match actual code. Detects "Code Leading Docs" drifts and feeds findings back to drive plan revisions.
  Last Modified: 2026-05-06 11:20:00 (CST)
-->
# Stage 4 — Doc-Code Consistency Sync (Built-in)

> Purpose: Periodically or on-demand verify consistency between design documents and actual code; detect "Code Leading Docs" deviations (drifts) and feedback results to drive `plan` revisions.

## Design Philosophy

This stage is not delegated to an external tool; it is a built-in step of `doc-driven-dev`. **Mappings are unified in `document_inventory[]` within `{METADATA_DIR}/project-schema.json`**, allowing verification whether or not documents carry a metadata header.

### Information Sources

```
{METADATA_DIR}/project-schema.json
    │
    ├─ scope.active/maintenance_versions    → Determine scan range
    ├─ document_inventory[]                   → Parallel sync per code_paths
    │    └─ (Verify consistency between header and inventory if location=hybrid/embedded)
    └─ code_conventions.roots / ignore_patterns → Scan meta-rules
```

### Impact of `metadata_location` on Sync

| metadata_location | Sync Behavior |
|:---|:---|
| `external` | Fully dependent on `document_inventory`; does not read file headers. |
| `embedded` | Reads file header metadata first; verifies consistency between header and inventory cache; Mismatch → Drift. |
| `hybrid` | Prioritizes file header if present; falls back to inventory otherwise. |

### Drift Determination Principles

**Tolerance for Doc Leading Code; NO tolerance for Code Leading Doc.**

| Scenario | Determination |
|:---|:---:|
| Doc describes Feature X, Code not implemented | ✅ Non-drift (Normal state of doc-first) |
| Code implements Feature Y, Doc not describing it | ❌ Drift |
| Doc marks `not_started` / `partial`, Code partially implemented | ✅ Non-drift |
| `code_paths` referenced in doc do not exist | ⚠️ Out of Sync (Categorized as Drift) |
| Paths in Doc §8 Planning do not exist but `implementation_status=implemented` | ❌ Drift |

---

## Trigger Modes (Three Modes)

| Mode | Trigger | Scope |
|:---|:---|:---|
| **Incremental Sync** | Completion of each T-XXX in Stage 3 | Files involved in T-XXX + corresponding design doc. |
| **Status View** | Session start / User asks "Progress" | Read state file; no scanning. |
| **Full Sync** | Weekly / Phase milestone / Manual trigger | All design docs + corresponding code. |

---

## Configuration & State

### Config File: `{METADATA_DIR}/config.json`

(No longer maintained as a separate `doc-sync-config.json`)

```json
{
  "project_name": "...",
  "plan_file": "Docs/.../design-plan.md",
  "design_doc_roots": ["Docs/...", "docs/"],
  "references": { ... },
  "sync": {
    "enabled": true,
    "check_types_default": ["file_exists", "metadata_code_paths"],
    "ignore_patterns": ["**/*.test.rs", "**/target/**"],
    "drift_severity_rules": {
      "code_without_doc": "P0",
      "code_path_missing": "P1",
      "adr_violation": "P0",
      "term_mismatch": "P2"
    }
  }
}
```

### State File: `{METADATA_DIR}/state.json`

Combines four types of data: Planning Progress + Task Status + Drift List + Session Logs.

```json
{
  "revision": 42,
  "last_check": "2026-05-05T20:30:00Z",
  "current_stage": "develop",
  "current_phase": "I-3",
  "current_goal": {
    "goal_id": "G-v10-industry-standard-align",
    "title": "V10 alignment with industry-standard skill loading mechanism",
    "type": "feature_alignment",
    "reference_baseline": "/path/to/industry-standard",
    "expected_outcome": "...",
    "constraint": "Do not modify v{N} released APIs"
  },
  "design_docs": {
    "awesome-app-v8-phase-i-session-lock": {
      "path": "Docs/.../Phase_I-...md",
      "status": "approved",
      "score": 8.5,
      "last_updated": "2026-05-05",
      "code_paths_verified": true
    }
  },
  "authoring_checkpoints": {
    "awesome-app-v10-phase-a-session-lock": {
      "current_section": "§4",
      "sections_done": ["§0", "§1", "§2", "§3"],
      "sections_pending": ["§4", "§5", "§6", "§7", "§8"],
      "last_updated": "2026-05-05T22:10:00Z",
      "review_level": "L2"
    }
  },
  "review_sessions": [
    {
      "id": "rs-20260505-0001",
      "doc_id": "awesome-app-v10-phase-a-session-lock",
      "level": "L2",
      "models_used": ["claude-opus-4", "claude-sonnet-4"],
      "prompt_versions": { "design_refine": "1.0", "multi_model_review": "1.0" },
      "rounds": 2,
      "converged": true,
      "timestamp": "2026-05-05T21:30:00Z"
    }
  ],
  "dev_tasks": {
    "T-001": { "status": "done", "design_doc": "...", "commit": "abc123" }
  },
  "drifts": [
    {
      "id": "drift-001",
      "severity": "P1",
      "type": "code_without_doc",
      "code": "crates/awesome-app/src/foo.rs:42-58",
      "doc": "awesome-app-v8-phase-i-session-lock",
      "doc_section": "§3",
      "description": "...",
      "detected": "2026-05-05",
      "fixed": false
    }
  ],
  "session_changes": [
    {
      "id": "sc-20260505-0042",
      "at": "2026-05-05T22:10:00Z",
      "stage": "2.1",
      "action": "append_section",
      "target": "docs/phases/session-lock.md",
      "target_section": "§3",
      "summary": "drafted §3 component decisions",
      "undo_hint": {
        "kind": "text_revert",
        "before_sha256": "...",
        "after_sha256": "..."
      },
      "reverted": false
    }
  ],
  "next_steps": []
}
```

**Field Sources & Read/Write Rules**:
- `revision` — CAS check before all write operations (per `SKILL.md §Data Protocols`).
- `current_goal` — Written by `workflows/01-planning.md §Step 0.1`.
- `authoring_checkpoints` — Appended by `workflows/02-design-authoring.md` for each chapter.
- `review_sessions` — Appended upon completion of `prompts/multi-model-review.md §0.5`.
- `session_changes[]` — Every action causing file/JSON change MUST write an entry with `undo_hint`; consumed by `workflows/99-rollback.md`.

---

## Sync Types (5 Built-in Types)

| Type | Input | Logic |
|:---|:---|:---|
| `file_exists` | `inventory[].code_paths[]` | Verify if files actually exist. Drift if missing while `implementation_status=implemented`. |
| `inventory_code_paths` | `inventory[].code_paths` | Verify path existence + consistency between `implementation_status` and actual code status. |
| `schema_match` | `doc §8` + Struct/ORM fields | Compare doc-defined fields with actual code definitions (Name / Type / Constraints). |
| `api_match` | `doc §3 / §8` + Route registration | Compare doc endpoints with actual registered paths/methods. |
| `pattern_check` | Doc keywords + Code Grep | Implementation mentioned in doc (e.g., "Use tokio::Mutex") must appear in code; consumes `schema.glossary[]` to detect `term_mismatch`. |
| `template_out_of_date` | Doc `template_version` vs current skill version | Soft warning (not drift); prompts migration pass (see `SKILL.md §Template & Prompt Versioning`). |
| `dependency_inversion` | `inventory[].depends_on` + `status` | Warning/Block if child doc is `approved` while parent is still `draft`. |
| `constraint_violation` | Doc body + `schema.project_constraints[]` | Keyword scanning for violations (Positioning / Reference Usage / Scope Exclusion); soft warning for human review. |

Sync types are auto-selected by `config.sync.check_types_default` + inventory heuristics:
- `code_paths` non-empty → execute `inventory_code_paths`.
- Doc §8 contains data structure definitions → add `schema_match`.
- Doc contains API tables → add `api_match`.

---

## Execution Flow

### Mode 1: Incremental Sync (After each Stage 3 Task)

1. Read `design_doc` (doc_id) and `commit` file list for T-XXX.
2. Find `code_paths` for this doc_id in `project-schema.document_inventory`.
3. Run `file_exists` + `inventory_code_paths` for these files.
4. Add `schema_match` if Doc §8 data structures are involved.
5. Output drifts related ONLY to this task.
6. Clean result → Mark T-XXX `done`; P0/P1 drifts → Block completion.

### Mode 2: Status View (Quick Response)

1. Read `{METADATA_DIR}/state.json`.
2. Output in following format without scanning:
   ```
   Current Stage: [current_stage] / [current_phase]
   Last Check   : [last_check]
   Known Drifts : [total] (P0: X, P1: Y, P2: Z)
   Unfixed      : [unfixed]
   Next Step    : [next_steps[0]]
   ```

### Mode 3: Full Sync

1. Read `project-schema.json`, filter versions by `scope` (skip `archived`).
2. Traverse matching `document_inventory[]` entries:
   - Determine authoritative metadata per `metadata_location`.
   - Select sync types per `check_types_default` + doc features.
   - Execute in parallel (Parallelize all `Grep / Glob` calls).
3. Aggregate drifts into `state.json`.
4. Fixed drifts (those that pass re-sync) are marked `fixed: true`, preserving audit trace.
5. If new docs (unregistered .md) are found → Prompt to run `00-adoption.md` refresh.

---

## Output Format

Two distinct sections, **strictly separated**.

### Section 1: Phase Milestone Progress

```
Stage 3 Current Progress (Answer: "Where are we in the plan?")

Phase I-1: SessionWriteLock ✅ done (7/7 tasks)
Phase I-2: TranscriptRepair ✅ done (4/4 tasks)
Phase I-3: JSONL Auto-Repair ⏳ not_started (0/5 tasks)
```

### Section 2: Drift List

```
| drift-id | Severity | Type | Doc / Code | Description |
|:---|:---:|:---|:---|:---|
| drift-001 | P0 | code_without_doc | crates/.../foo.rs ↔ awesome-app-v8-phase-i | foo.rs has new `compact_sessions` fn, not described in doc §3. |
```

---

## Drift Handling Decision Tree

```
Drift Type
  │
  ├── code_without_doc
  │     └── Code implemented capability not mentioned in doc.
  │         ├─ Capability within planned slice → Supplement doc §3 + §8.
  │         └─ Capability outside plan → Escalate to Stage 2.2 Optimization to re-evaluate architecture.
  │
  ├── code_path_missing
  │     └── Doc declares implemented but file not found.
  │         ├─ Code deleted → Update doc §8 + implementation_status.
  │         └─ Path error → Fix doc code_paths.
  │
  ├── schema_mismatch
  │     └── Field definitions deviate from doc.
  │         ├─ Doc wrong → Fix Doc.
  │         └─ Code wrong → Fix Code.
  │
  ├── api_mismatch
  │     └── Endpoint definitions deviate from doc (Similar to schema).
  │
  └── term_mismatch
        └── Terminology / Naming inconsistency → Fix Doc (Usually).
```

---

## State Maintenance Rules

| Action | Trigger | Result |
|:---|:---|:---|
| **Add Drift** | Mismatch found during sync. | Append to `drifts[]`, `fixed: false`. |
| **Mark Fixed** | Next sync passes at same location. | Update `fixed: true` + `fixed_at`. |
| **Do NOT Delete** | — | Keep fixed items for audit. |
| **Escalate Severity**| Drift remains unfixed for N days. | P2 → P1 → P0 (Optional, config-driven). |
| **Task Completed** | Stage 3 task marked done. | Update `dev_tasks[T-XXX].status` and `commit`. |

---

## Closing the Loop with Plan

If full sync reveals flaws in the design plan itself:

1. Add issue entries to `design-plan.md §1.3` (A/B/C/D classification).
2. Record feedback in `design-plan.md §5` execution log.
3. Update §2.2 batch table if scheduling is affected.
4. Point `state.json.next_steps` to the documents for the next optimization round.

---

## Out-of-the-Box Prerequisites

No independent `init` command. Before starting, ensure:

1. `{METADATA_DIR}/project-schema.json` exists and `document_inventory[]` is non-empty.
   - If not met → Auto-redirect to `workflows/00-adoption.md`.
2. `{METADATA_DIR}/config.json` exists.
   - If missing → Created during `00-adoption.md`.
3. `{METADATA_DIR}/state.json` exists.
   - If missing → Create empty state and immediately run first full sync.

> As long as adoption is complete and inventory is registered, sync is "out-of-the-box." No changes required for existing docs; new docs can optionally carry metadata headers.

---

## Constraints

1. **No Manual Editing of Drift List**: Only append or mark fixed via the sync process.
2. **Do NOT Delete Fixed Drifts**: Preserved for audit trail.
3. **P0 Drifts MUST be Handled in Session**: Or explicitly postponed to next task and registered.
4. **Parallelize Grep/Glob**: Sync duration should not block development.
5. **Full Sync NOT Interruptible**: Must either complete or rollback state to last successful point.
