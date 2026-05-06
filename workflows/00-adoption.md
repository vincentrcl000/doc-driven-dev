<!--
  Purpose: Workflow for Stage 0 — First-time Activation / Project Adoption.
  Function: Generates project-schema.json and base config; auto-adapts to Greenfield (new) or Brownfield (existing) projects. Includes idempotent Path R (Refresh) logic.
  Last Modified: 2026-05-06 12:25:00 (CST)
-->
# Stage 0 — First-time Activation / Project Adoption

> Purpose: **Activate** `doc-driven-dev` for the first time in a project. Generates `project-schema.json` and base config; auto-adapts to Greenfield (new) or Brownfield (existing) scenarios.

## When to Use

**MUST** be run in any of the following scenarios:
- `{METADATA_DIR}/project-schema.json` does not exist.
- User says: "Enable doc-driven-dev in this project" / "Adopt existing project docs."
- User enters other Stages (1/2/3/4) but no schema is detected → Auto-redirect to this workflow.

**Do NOT skip**. All subsequent Stages rely on the `project-schema.json` produced by this workflow.

---

## Three Paths

```
Run 00-adoption.md
     │
     ├─ Step 0: Resolve {METADATA_DIR} (Host Detection)
     │
     ├─ Step 0.5: Idempotency Check (Decide Fresh vs. Refresh)
     │
     ├── Schema/Plan already exists
     │      └──▶ 【Path R: Refresh】
     │              Supplement gaps only; never overwrite existing outputs.
     │
     ├── Project Docs root is empty or ≤ 2 documents
     │      └──▶ 【Path A: Greenfield】
     │              Create initial structure + empty schema per spec.
     │
     └── Project has ≥ 3 existing .md design docs
            └──▶ 【Path B: Brownfield】
                    No intrusion on user docs; discovery & registration to schema.
```

All three paths produce the same structured `project-schema.json`. Path R is triggered only when adoption traces exist, ensuring re-runs do not destroy existing files.

---

## Step 0 — Host Detection (Common to all paths)

Determine `{METADATA_DIR}` per `SKILL.md §Metadata Directory Resolution`:
1. Check project root for `.claude/` → Candidate `.claude/doc-driven-dev/`.
2. Check project root for `.codex/` → Candidate `.codex/doc-driven-dev/`.
3. Both exist → `AskUserQuestion` to choose primary host (persist to `config.json.host`).
4. Neither exists → `.doc-driven-dev/`.

**Immediate Outputs**:
- Create `{METADATA_DIR}/` (if missing).
- Pre-fill `config.json` with host/metadata info.

**Constraints**:
- One project MUST NOT use two `{METADATA_DIR}` locations (prevents state split).
- If host host changes in future sessions, respect `config.json.host` but prompt user if mismatch detected.
- Never write to `.omc/` or `.omx/` (private domains of external orchestrators).

---

## Step 0.5 — Idempotency Check (Path Decision)

Detect existing artifacts in order; **any match switches to Path R**:

| Detection Item | Path |
|:---|:---|
| schema | `{METADATA_DIR}/project-schema.json` |
| config | `{METADATA_DIR}/config.json` (with fields beyond host/dir) |
| design-plan | `<docs_root>/design-plan.md` or `plan_file` in schema |
| dev-plan | `<docs_root>/dev-plan.md` |
| inventory | `schema.document_inventory` is non-empty |

**Decision Rules**:
- All missing → Diverge to Path A / Path B based on doc count.
- Any exists but schema/inventory is empty → Path R "Gap-filling" sub-mode.
- Schema exists and inventory is non-empty → Path R "Refresh" sub-mode.

**Enforced Principle**: In Path R, **NEVER** overwrite:
- `<docs_root>/design-plan.md` / `dev-plan.md`.
- Any registered design doc in `document_inventory[]`.
- Existing fields in `project-schema.json` (Append/Mark only, no rewrite).

---

## Path R — Refresh (Idempotent Sync)

> Use Case: Re-run / Partial initialization / User-triggered "Refresh Adoption."

### R.1 Output Reconciliation Table

```
Adoption Status ({METADATA_DIR}):
  ✅ project-schema.json (inventory 33 items, last_synced 2025-04-12)
  ✅ config.json
  ⚠️ state.json missing → Will create empty skeleton
  ✅ docs/design-plan.md
  ❌ docs/dev-plan.md missing
```

### R.2 Supplement (Create missing artifacts only)

For each `❌ Missing` item:
- Plan files → Create skeleton from templates; metadata `status: not_started`.
- `state.json` → Write minimal empty structure.
- `config.json` → Write base version per `01-planning.md §Step 6`.

**Existing files are strictly untouched.**

### R.3 Incremental Discovery (Brownfield sub-mode only)

If `document_inventory` is empty or last scan > 7 days:
1. Re-run `Path B §B.1-B.5` discovery, but **only consider documents NOT in current inventory as new candidates**.
2. Present new items to user for confirmation before appending.
3. Do NOT delete existing inventory entries (mark moved/renamed files as `status: stale`).

### R.4 Schema Field Refresh

Only these fields allow auto-updates:
- `last_synced_at`.
- Appended `document_inventory[]` entries.
- Appended `roadmap_references[]` entries.

Changes to other fields (`project_name`, `scope`, `conventions`) require `AskUserQuestion` authorization.

---

## Path A — Greenfield

### A.1 Interactive Collection

Via `AskUserQuestion`:
1. **Project Name** / **Doc root suggestion** (Default `docs/`).
2. **Tech stack language** (for `code_conventions`).
3. **Reference Projects** (Local paths / URLs, optional).
4. **Topic of first design doc** (Guide user intent).

### A.2 Create Directory Structure (Idempotent)

Check for existence via `Glob` before creation. **Skip if already exists**:
```
docs/                       (Skip if exists)
├── design-plan.md          (Skip if exists; else copy from templates)
├── dev-plan.md             (Skip if exists; else copy empty skeleton)
└── phases/                 (Skip if exists; else create empty dir)
```

### A.3 Write Schema (Greenfield Defaults)

Per `schema/project-schema.greenfield.example.json`:
- `project_type: "greenfield"`
- `metadata_location: "embedded"` (New docs carry headers)
- `document_inventory: []` (Empty initially)

### A.4 Write Config (Workflow Preferences)

Create base `{METADATA_DIR}/config.json`.

---

## Path B — Brownfield (Existing Projects)

### B.1 Discover Documents

1. Confirm **Doc Root** (auto-suggested if obvious candidate exists).
2. List `.md` files via Glob.
3. If > 100 docs → Warn user to "define scope first."

### B.2 Identify Naming Conventions

Regex extraction from subdirectories:
- Version grouping: `V\d+` / `v\d+`.
- Phase grouping: `Phase [A-Z]`.
- Roadmap pattern: `*Roadmap*.md`.
- Design spec pattern: `*Phase*Design*Spec*.md`.

Display inferred results to user for confirmation/correction.

### B.3 Interactive Scope Determination

List all discovered versions:
```
Discovered versions: V2, V3, ..., V10
Classify (multiple choice):
  [ ] active (current iteration, involved in all stages)
  [ ] maintenance (bug fixes + sync only)
  [ ] archived (skip scanning)
```

### B.4 Identify Roadmaps

For each `active` + `maintenance` version, find matches for `roadmap_pattern`:
- Single match → Append to `roadmap_references[]`.
- Zero/Multiple matches → Ask user.
Roadmaps marked `treated_as: "design_plan"` to bypass Stage 1 regeneration.

### B.5 Register Document Inventory

For every match of `design_doc_pattern` in managed versions:
1. **Parse from filename**: Extract `version`, `phase`, `topic` via `naming_template`.
2. **Infer doc_id**: `{project-slug}-v{version}-phase-{phase-lower}-{topic-slug}`.
3. **Supplement from roadmaps** (if possible): Extract `implementation_status` from phase tables.
4. **Infer code_paths**: Grep for paths like `src/` or `crates/` in content.
5. Mark `derived_from: "filename"` or `"hybrid"`.
Leave unknown fields (`score`, `owner`, `last_updated`) as null for future filling.

**Do NOT modify any user documents**. All info is externalized to the schema.

### B.6 Fixation of Adoption Rules

Ask user: "Use embedded metadata headers for **newly created** docs?"
- Yes → `metadata_location: "hybrid"`.
- No → `metadata_location: "external"`.

### B.7.5 Create Progress Dashboard (Mandatory for Brownfield)

Even if a roadmap is registered as `design_plan`, a dedicated `<docs_root>/design-plan.md` **MUST** be created as a **Progress & Scoring Dashboard**.

| File | Responsibility | Maintenance |
|:---|:---|:---|
| `roadmap_references[*].file` | **Design Intent Source**: Phased plans, goals. | Manual by User. |
| `<docs_root>/design-plan.md` | **Dashboard**: 6D/4D scores, status, audit log. | Auto-updated by skill Stages 1-4. |

### B.8 Diagnostic Report

Display "Project Profile" summary to user:
```
✅ Brownfield Adoption Complete — AwesomeApp
Documents: 87 total (V2-V10)
Active (V10): 11 docs | Maintenance: 22 docs | Archived: 54 docs (skipped)
Roadmaps: 3
Registered in Inventory: 33
Inference Completeness:
  ✅ Has code_paths: 8 docs
  ⚠️ code_paths empty: 25 docs (Run Stage 4 Sync to supplement)
  ⚠️ status unknown: 18 docs
```

---

## Constraints

1. **No modification of existing user docs**: Path B is read-only; info is externalized.
2. **No auto-determination of Scope**: Classifying active/maintenance/archived requires user decision.
3. **No forced roadmap**: If none exist, Stage 1 generates a plan as normal.
4. **Idempotency First**: Existence checks before any write; use Path R logic if artifacts found.
5. **Traceability**: Every inventory entry carries a `derived_from` field for auditing.
