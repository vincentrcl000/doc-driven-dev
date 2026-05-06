<!--
  Purpose: Prompt template for Development Review (Code-Doc Alignment).
  Function: Provides a self-check checklist for use after task completion and before submission; ensures zero deviation between implementation and design documents.
  Last Modified: 2026-05-06 11:35:00 (CST)
-->
# Prompt: Development Review (Code-Doc Alignment Self-Audit)

> Purpose: A self-audit checklist for use after task completion and before submission; ensures zero deviation between implementation and design documents.

---

## Role

You are a strict code reviewer. Your objective is to **verify that the current implementation is fully aligned with the design document**. Any deviation must have a clear handling path (Change Code / Change Doc / Register Drift).

---

## Inputs

```
[Task ID]: T-XXX
[Corresponding Design Doc]: [doc-id + path]
[Referenced Section]: §X.Y (from dev-plan.md)
[Files involved in this change]: [List of paths]
[New / Modified Tests]: [Test file paths + Test names]
```

---

## Self-Audit Checklist

### A. Implementation Alignment

- [ ] **A1. Component Responsibility**: Responsibilities of new/modified code are consistent with the component table in design §2.2.
- [ ] **A2. Key Decisions**: Core implementation choices fully align with the decision tables in design §3 (no unexplained deviations).
- [ ] **A3. File Paths**: Code paths match the file planning in design §8; any deviation has been registered as a drift.
- [ ] **A4. Dependencies**: No new dependencies (crates, libraries, infrastructure) introduced that weren't mentioned in design §0.2 / §5 / §8.
- [ ] **A5. Data Model**: Data structures fully align with fields, types, and constraints described in the design.

### B. Edge Cases

- [ ] **B1. Risk Coverage**: Every risk marked as "Mitigated ✅" in design §6 is indeed handled in this implementation.
- [ ] **B2. Unidentified Risks**: New edge cases discovered during development have been written back to design §6 (or recorded in dev-plan notes for future write-back).
- [ ] **B3. Error Handling**: Error handling paths are consistent with the design (no silent swallowing, no false successes).

### C. Testing

- [ ] **C1. Acceptance Mapping**: Every `[ ]` acceptance criterion in design §7 has a corresponding test case.
- [ ] **C2. Executable Commands**: Test commands match design §7 and pass locally.
- [ ] **C3. No Bypassing**: No "faking a pass" by lowering assertion strength, commenting out tests, or stubbing dependencies.

### D. Document Write-back

- [ ] **D1. Implementation Status**: Status of the corresponding slice in design §0 has been updated (`not_started` → `partial` or `implemented`).
- [ ] **D2. File Planning**: Corresponding entries in §8 have been moved from "Planned New" to "Created."
- [ ] **D3. Acceptance Toggling**: Passed items in §7 have been changed from `[ ]` to `[x]`.
- [ ] **D4. Metadata Bump**: `version` in the header has been incremented; `last_updated` has been updated.
- [ ] **D5. Discrepancy Log**: If a design decision was overruled during implementation, a discrepancy entry has been added to §0.3.

### E. Development Plan

- [ ] **E1. Task Status**: T-XXX status in `dev-plan.md` has been updated.
- [ ] **E2. Commit Association**: Commit message contains `T-XXX`.
- [ ] **E3. Implementation Notes**: Section §3 "Implementation Notes" for T-XXX in dev-plan has been supplemented with key points.

### F. Sync Verification

- [ ] **F1. Incremental Sync Executed**: Incremental sync has been performed per `workflows/05-sync.md` Mode 1.
- [ ] **F2. No New P0/P1 Drifts**: No high-severity drifts introduced (or handling path has been decided).
- [ ] **F3. Mark Fixed Drifts**: Existing drifts fixed in this change have been marked `fixed: true` in `{METADATA_DIR}/state.json`.

---

## Output

Provide a determination for each item:

```
A1. ✅ / ❌ + Evidence (Referencing code or doc section)
A2. ...
...
```

**All ✅** → Ready for submission.
**Any ❌** → Provide fix suggestions; re-review after rework.

---

## Red Flags (STOP Submission Immediately)

The following are considered serious violations and must be addressed immediately:

1. Code implements capabilities NOT described in the design document (undocumented implementation).
2. Tests lower assertion strength to pass (e.g., changing `assert_eq!(x, 5)` to `assert!(x > 0)`).
3. An original acceptance criterion in design §7 is commented out or deleted.
4. New file paths are not in §8 planning and have not been registered as drifts.
5. An `approved` design document is modified silently (no version bump, no `in_review`).

---

## Relationship with Full Sync

This self-audit focuses on **precise alignment of current changes**; Full Sync (`workflows/05-sync.md` Mode 3) focuses on **consistency of the entire system**. They are complementary:

- Passing this self-audit ≠ Passing Full Sync (Historical drifts may still exist).
- Passing Full Sync ≠ Passing this self-audit (Current changes may have unsynced deviations from the doc).

Both must pass before status can be `done`.
