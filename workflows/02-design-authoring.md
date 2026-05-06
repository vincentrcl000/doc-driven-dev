<!--
  Purpose: Workflow for Stage 2.1 — Design Document Authoring.
  Function: Guides the user in drafting high-quality design documents from scratch or filling existing skeletons, section-by-section.
  Last Modified: 2026-05-06 10:50:00 (CST)
-->
# Stage 2.1 — Design Document Authoring

> Purpose: Author a high-quality initial draft (or fill out the framework) for a single design document from 0 to 1.

## When to Use

- When the design plan points to a document that needs to be newly created (`status=draft` or non-existent).
- When the user explicitly says "Write a design spec for [Topic]."

## Inputs

From `{METADATA_DIR}/config.json` or provided by the user:

1. **Target Document Info**: `doc_id` / Title / Associated Phase / Dependent Docs.
2. **Reference Sources**: Automatically read from config; guide user to supplement if new sources are available this round.
3. **Benchmark Document** (Critical): An existing high-quality document from the same project used to benchmark structure and depth.
4. **Implementation Status** (If already started): Existing code paths, passed tests.

---

## Core Principles

1. **Section-by-Section Authoring**: Prohibit generating the entire document at once; write only 1-2 sections per round, moving to the next only after user confirmation.
2. **Benchmark-Driven**: Before starting each chapter, explicitly state "This chapter will achieve the depth of [Benchmark Doc] §X."
3. **Guided References**: At the end of each chapter, proactively ask "Would you like to supplement the implementation details of [Capability] from a certain reference project?"
4. **No Faked Code Paths**: Must use `Grep/Read` to verify the actual existence of any `code_paths` before referencing them.
5. **Skeleton First**: Use `templates/design-doc.md` in the first round to fill a complete skeleton (all chapter titles + TODO markers), then refine chapter by chapter in subsequent rounds.

---

## Process Steps

### Step 0 — Load Checkpoint (Session Resume)

Before reading the target file, check `{METADATA_DIR}/state.json.authoring_checkpoints[doc_id]`:

| State | Behavior |
|:---|:---|
| No entry | First-time authoring; proceed to Step 1 Full Initialization. |
| Entry exists and `sections_pending` is non-empty | **Resume from Checkpoint**: Skip Step 1, directly resume the Step 2 chapter loop from `current_section`; playback the list of completed chapters and previous notes to the user. |
| Entry exists but `sections_pending` is empty | Document is fully authored; skip to Step 3 Self-check. |

Brief the user before entering Step 2:

```
📘 Resuming Authoring: awesome-app-v10-phase-a-session-lock
  Completed: §0, §1, §2, §3
  Current Chapter: §4
  Last Note: pending review on §3 decision table
```

### Step 1 — Skeleton Initialization (Idempotent)

1. **Existence Check**: First `Glob` the target path `[Project]/[Phase]-[Topic].md`.
   - **Exists** → Do NOT copy template. Instead:
     - `Read` current file, compare against template chapter list.
     - Missing chapters → `Edit` to append (preserving existing body).
     - Existing chapters → Skip, proceed to Step 2 to resume.
     - Record "resume authoring on <path>" in `state.json.session_changes[]`.
   - **Does Not Exist** → Copy `templates/design-doc.md` to the path.
2. Fill/Verify Metadata Header (per `templates/doc-metadata-schema.md`); for existing metadata, only `diff` missing fields; do not rewrite `version` or `status`.
3. Keep all chapter titles; use `> TODO` as placeholders for the body (only for new/missing chapters).
4. Show the skeleton or current status to the user; confirm before entering Step 2.

**Prohibited**: Direct `Write` overwriting an existing design draft file.

### Step 2 — Section Filling

Fill each chapter in order (recommended §0 → §1 → §2 → §3 → §4 → §5 → §6 → §7 → §8):

#### Per-Chapter Cycle

1. **Read Benchmark Section**: Open the corresponding section in the benchmark doc and summarize its structure and depth.
2. **Consult Reference Sources**:
   - If local reference projects are configured, `Grep / Read` relevant files in their directories.
   - Use `prompts/reference-extraction.md` to extract key implementation takeaways.
3. **Draft Current Section**:
   - **§0 Implementation Status**: Must be specific; if none exists, explicitly write `not_started`.
   - **§2 Layered Architecture**: Use Mermaid or ASCII diagrams; categorize component tables by "Implemented / Planned."
   - **§3 Core Components**: **Use Decision Tables** (Decision / Choice / Reason); prohibit pseudocode blocks > 10 lines.
   - **§6 Key Risks**: Each item must carry a status marker (✅/⚠️/🔜) + target slice.
   - **§7 Acceptance Criteria**: Strictly distinguish between `[x]` (completed) and `[ ]` (planned).
   - **§8 File Planning**: Four clear categories (Created / Planned New / Planned Mod / Not Introduced).
4. **Confirmation & Iteration**: Show the chapter to the user; if a discrepancy is noted, revise before moving to the next.
5. **Update Metadata**: Bump `version` when the chapter is complete; write `template_version` (current skill template version) on first creation.
6. **Update Checkpoint**: Write to `state.json.authoring_checkpoints[doc_id]`:
   - Move current section from `sections_pending` to `sections_done`.
   - Point `current_section` to the next chapter.
   - Use current time for `last_updated`.
   - Note "what to do next time" in `notes`.
   - CAS bump state `revision`.
   - Append an entry to `session_changes[]` with `undo_hint` for rollback support.

### Step 3 — Self-Check

After completion, run the 12-point audit from `prompts/design-refine.md`. If any fail, return to Step 2 to revise the corresponding chapter.

### Step 4 — Enter Optimization Phase

1. Mark document `status=in_review`.
2. When document reaches `status=approved`, delete the doc_id from `state.json.authoring_checkpoints` (preserve history in `session_changes[]`).
3. Recommend to the user:
   - Single-model optimization: Directly call `workflows/03-design-optimization.md`.
   - Multi-model review: Follow the Step 1-9 process in `prompts/multi-model-review.md`.

---

## FAQ & Troubleshooting

| Scenario | Handling |
|:---|:---|
| User asks for "Full generation" at once | Refuse and explain: Section-by-section avoids cumulative errors; I will guide you chapter by chapter. |
| Reference project code is too massive to read | Use `prompts/reference-extraction.md` for targeted extraction of key implementation points. |
| Benchmark doc quality is questionable | Remind user to revise the benchmark doc first, or switch to a finalized doc. |
| Unsure about a design decision | Present 2-3 candidate options with a comparison table; let user decide. |
| User requests pseudocode | Allowed but restricted: ≤ 10 lines per block; exceeding must be converted to a Decision Table. |

---

## Constraints

1. **No Unilateral Architecture Decisions**: Choosing between multiple options must be decided by the user.
2. **No Omitted Metadata**: Missing metadata header is considered incomplete.
3. **No Copying Reference Code**: Only benchmark concepts and decisions; do not paste code.
4. **No Marking ✅ for Unimplemented Items**: Documents with `status=draft` default to `implementation_status=not_started` for all items.
5. **No Proceeding Before Completion**: Do not move to the next chapter until the current one is finished to avoid metadata versioning confusion.
