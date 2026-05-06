<!--
  Purpose: Workflow for Stage 3 — Development Execution (Implementation).
  Function: Uses finalized design documents as the sole specification to organize code development, manage task progress, and ensure traceability of every commit back to design decisions.
  Last Modified: 2026-05-06 11:15:00 (CST)
-->
# Stage 3 — Development According to Design

> Purpose: Use the approved design document as the single source of truth for code development, task management, and ensuring every commit is traceable back to a design decision.

## When to Use

- When design documents have reached `status=approved`.
- When a `dev-plan.md` exists or is about to be generated.
- When the user asks to "Implement [Capability] according to [doc-id]."

## Core Disciplines

1. **Document First**: Before any code change, the design document must already describe that change; otherwise, go to Stage 2 to supplement the document first.
2. **Task as Deliverable**: Every development task must correspond to an entry in `dev-plan.md`, including `doc_id + section`.
3. **Test Binding**: Task test criteria directly reference design spec §7; do not write separately.
4. **Deviance as Drift**: When implementation deviates from design, register immediately and decide (Change Doc vs. Change Code).
5. **No Implicit Expansion**: Task scope strictly aligns with the design doc; changes outside the scope require new tasks.

---

## Execution Steps

### Step 1 — Pre-task Check

When receiving a request to "Implement T-XXX":

1. Read the T-XXX entry in `dev-plan.md`.
2. Read the referenced section (§X.Y) of the corresponding design document.
3. Check sync status (Mode 2 of `workflows/05-sync.md`, read `{METADATA_DIR}/state.json`):
   - If files involved in this task have unpatched drifts → Resolve drifts before starting.
4. Proceed to Step 1.5 after user confirmation of understanding.

### Step 1.5 — Code Reality Probe (Mandatory Pre-encoding)

> Purpose: Avoid blind encoding based on outdated designs. Calibrate design assumptions with actual code status before writing.

Execution Checklist:

1. **Path Verification**: Run `Glob` for each path listed in design §8 (File Planning).
   - Paths in `Created` category must exist; missing → Register drift `code_path_missing`.
   - Paths in `Planned New` category should not exist; exists → Register drift `code_already_exists`.
2. **Key Signature Comparison**: For core components listed in design §3:
   - `Read / Grep` corresponding `struct / class / fn / interface` in actual code.
   - Compare: Method names, parameter lists, return types, key fields.
   - Mismatch → Register drift `signature_mismatch`, noting expected vs. actual.
3. **Dependency Status**: Check if external crates/packages mentioned in design §5 / §8 are present in `Cargo.toml / package.json`; missing → drift `dependency_missing`.
4. **Test Status**: `Grep` test files corresponding to design §7 acceptance criteria.
   - Existing tests → List current pass/fail status as baseline.
   - No tests → Mark as mandatory addition for this task.

**Summarize Probe Results** for the user:

```
🔍 T-XXX Code Reality Probe
  Design §8 File Planning: 5 items
    ✅ Consistent with reality: 3 items
    ⚠️ Drift: 2 items
       - crates/foo/src/bar.rs (Planned new but already exists with 12 lines of impl)
       - crates/foo/src/baz.rs (Design §3 expects fn handle(), reality is fn process())
  External Deps: tokio ✅ / serde ✅ / Missing: tracing-subscriber
  Existing Tests: crates/foo/tests/bar_test.rs (3 pass / 1 fail)
```

**Decision Branch** (Must be decided by user, do not automate):

| Probe Finding | Options |
|:---|:---|
| Drift count = 0 | Proceed directly to Step 2 Encoding. |
| Drift ≤ 2 and non-blocking | User choice: Continue encoding and write back in Step 4 / Go back to Stage 2 to fix design. |
| Drift > 2 or involves core structure | **Mandatory** return to Stage 2.2 to optimize design first, then return to Step 1. |

All drift entries are written to `{METADATA_DIR}/state.json.drifts[]`, tagged with `discovered_at: "stage3_step1.5"`.

### Step 2 — Implementation

1. Implement code per design §3 (Component Design).
2. Handle key decisions per design §5 (Tech Details).
3. Write tests per design §7 (Acceptance Criteria).
4. Code paths must align with design §8 (File Planning); if they deviate, register drift.

### Step 3 — Self-Review

After encoding, self-review per `prompts/dev-review.md`:
- [ ] Implementation is exactly consistent with design §3 decision tables.
- [ ] Edge case handling covers identified risks in design §6.
- [ ] Test cases map to every acceptance item in design §7.
- [ ] File paths are exactly consistent with design §8.
- [ ] No new dependencies introduced that weren't mentioned in design.

### Step 4 — Document Write-back

If the implementation results in the following changes, the design document MUST be updated:

| Change | Section to Update |
|:---|:---|
| Actual code paths differ from §8 planning | §8 Created / Planned Mod |
| Unexpected edge cases discovered | §6 Key Risks (New items) |
| Acceptance criteria need supplement | §7 Acceptance Criteria |
| A decision found unviable after impl | §0.3 Key Design Changes (Discrepancy Log) |

Update and bump `version`; maintain `status` as `approved` or revert to `in_review` as appropriate.

### Step 5 — Update dev-plan

1. T-XXX status: `in_progress` → `in_review`.
2. Fill "Implementation Notes" (Template §3).
3. Record commit hash.
4. Update §4 Progress Tracking.

### Step 6 — Trigger Incremental Sync

Run `workflows/05-sync.md` Mode 1 (Incremental Sync):
- Limit scope to files involved in T-XXX + corresponding design doc.
- No new drifts → T-XXX can be marked `done`.
- New drifts → Return to Step 4 to supplement docs or follow decision tree; re-sync until clean.

---

## Discrepancy Handling Decision Tree

```
Code-Design Deviation Found
  │
  ├── Design error?
  │     └── Yes ─▶ Update Design (§0.3 Log) + bump version
  │
  ├── Code error?
  │     └── Yes ─▶ Fix code to align with design
  │
  └── Both have value?
        └── Escalate to Stage 2.2 (workflows/03-design-optimization.md)
            Decision via multi-model review
```

---

## State Data Consistency

All state generated during this stage is written to `{METADATA_DIR}/state.json`:

| Data | Field | Writing Timing |
|:---|:---|:---|
| Task Status / Commit | `dev_tasks[T-XXX]` | Step 5 |
| New Drift | `drifts[]` | Step 6 Auto-append |
| Session Changes | `session_changes[]` | Every chapter completion |
| Next Start Point | `next_steps[]` | After Step 6 completion |

**Do NOT maintain duplicate drift lists in dev-plan**; dev-plan §6 is a read-only view of `state.json.drifts`.

---

## Constraints

1. **No "Developing while changing design"**: Stop encoding to update design; follow Step 4 write-back.
2. **No Bypassing Tests**: Task cannot be `done` until all §7 acceptance items pass.
3. **No Implementation Without Design Basis**: Even if simple, check if the design doc mentions it.
4. **No Cross-task Mixing**: One commit per T-XXX; do not bundle multiple tasks.
5. **No Skipping Step 1.5 Probe**: Not a single line of code should be written before the probe and decision.
