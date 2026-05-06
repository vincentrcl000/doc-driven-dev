<!--
  Purpose: Workflow for Stage 99 — Rollback & Undo.
  Function: Provides a mechanism to undo the last N changes recorded in state.json.session_changes[].
  Last Modified: 2026-05-06 11:30:00 (CST)
-->
# Stage 99 — Rollback & Undo

> Purpose: Undo the current or last N file or JSON changes produced by the skill. Implemented based on structured logs in `state.json.session_changes[]` and `undo_hint`.

## When to Use

- When the user says "Undo last step" / "Roll back current optimization" / "undo."
- When the skill detects an error after writing (CAS conflict, validation failure) and needs to auto-rollback.
- When a multi-model review goes off-track and needs to return to a previous draft state.

## Prerequisites

- `{METADATA_DIR}/state.json` exists and contains `session_changes[]`.
- If the entry to be rolled back involves uncommitted Git state, remind the user to back up first (e.g., `git stash`).

## Rollable Action Types

Corresponding to `session_changes[].undo_hint.kind`:

| kind | Meaning | Rollback Strategy |
|:---|:---|:---|
| `text_revert` | Text edits made to an .md/.json file. | Verify current state with `before_sha256`, restore `before` content; if current SHA has changed (indicating manual post-edit) → Abort and report conflict. |
| `json_patch_revert` | JSON patch applied to schema / state / config. | Reverse apply the patch; protected by `revision` CAS; abort if revisions mismatch. |
| `file_created` | New file created. | Delete the file (after confirming no manual edits by the user). |
| `file_renamed` | Moved / Renamed. | Reverse rename; verify source path is not occupied. |

## Execution Process

### Step 1 — Determine Scope

Use `AskUserQuestion` to let the user choose:
1. Roll back the last 1 entry (Default).
2. Roll back the last N entries (User-specified N).
3. Roll back a specific `sc-id`.
4. Roll back all entries in the current session (filtered by `at` time + `stage`).

### Step 2 — Pre-check

For each `sc` entry to be rolled back:
- Verify prerequisites (SHA / Revision / File existence) based on `undo_hint.kind`.
- If any pre-check fails → Stop, display failure list to user, and require decision for each (Force rollback / Skip / Cancel batch).

### Step 3 — Sequential Rollback (LIFO)

Roll back strictly in reverse chronological order (`at`). For each success:
1. Set `session_changes[].reverted` to `true` and append `reverted_at` timestamp.
2. Append a new "inverse operation" `session_changes[]` entry (do NOT delete the original); `action` should be like `revert_of:sc-xxxx`.
3. CAS bump `state.revision`.
4. If schema changes are involved, also bump `schema.revision`.

### Step 4 — Report

```
🔙 Rolled back 3 changes:
  • sc-20260505-0042 (2.1 append_section docs/phases/session-lock.md §3) ✅
  • sc-20260505-0041 (2.1 append_section docs/phases/session-lock.md §2) ✅
  • sc-20260505-0040 (2.1 file_created docs/phases/session-lock.md) ✅

Affected Artifacts:
  • docs/phases/session-lock.md  Deleted

Recommended Next Steps:
  • If you need to restart this document, run workflows/02-design-authoring.md
```

---

## Non-rollable Scenarios

The skill MUST **refuse automatic rollback** and inform the user to handle manually in these cases:

| Scenario | Reason |
|:---|:---|
| `before_sha256` of original entry mismatches current file SHA. | Manually edited; auto-rollback would lose user changes. |
| CAS verification failure (`revision` mismatch). | Interference from concurrent sessions. |
| Entry `at` is earlier than the last `adoption refresh`. | Risk of rolling back across adoption boundaries is too high. |
| Entry already has `reverted=true`. | Idempotency protection to avoid double rollback. |
| Involves external artifacts already committed to Git (e.g., `crates/foo/*.rs`). | Rolling back to workspace loses commit history info; handle via Git. |

---

## Constraints

1. **NEVER directly delete `session_changes[]` entries**: Only flip the `reverted` flag + append inverse records; preserve full audit trail.
2. **CAS Protection**: All JSON rollbacks must pass the CAS protocol in `SKILL.md §Data Protocols → Concurrency`.
3. **No Cross-boundary Rollback**: Do not roll back across `adoption refresh` or across sessions in bulk; if requested, execute in batches limited to 10 entries.
4. **Post-rollback Next Steps decided by User**: Skill only executes rollback; does not auto-rerun any workflows.
