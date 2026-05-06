<!--
  Purpose: Specification for Project Schema (project-schema.json).
  Function: Defines how the skill perceives the project's structure, document inventory, and conventions. Serves as the metadata source for all automation.
  Last Modified: 2026-05-06 11:50:00 (CST)
-->
# Project Schema Specification

> **Schema-driven Core Philosophy**: The skill does not require user documents to carry metadata. Instead, it reads `{METADATA_DIR}/project-schema.json` to understand "what this project looks like." User documents, naming conventions, and directory structures do NOT need to be modified for the skill.

## File Location

```
{METADATA_DIR}/project-schema.json   (Committed, shared across the team)
```

## Division of Labor with Other JSON Files

| File | Nature | Typical Change Frequency |
|:---|:---|:---|
| `config.json` | Workflow preferences (host, sync rules, reference sources). | Rarely changes. |
| **`project-schema.json`** | **Project profile (Doc inventory, Version scope, Naming conventions).** | **Updated during version iterations / phase transitions.** |
| `state.json` | Dynamic state (Tasks, drifts, sessions). | Frequent changes; partially gitignored. |

---

## Top-level Fields

| Field | Type | Required | Description |
|:---|:---|:---:|:---|
| `schema_version` | string | ✓ | Version of this schema spec (Current: `1.1`). |
| `revision` | int | ✓ | Optimistic concurrency counter; +1 on every write; used for CAS. See `SKILL.md §Data Protocols`. |
| `project_type` | enum | ✓ | `greenfield` / `brownfield`. |
| `project_name` | string | ✓ | Name of the project (aligned with config). |
| `created_at` | date | ✓ | Date of schema initialization. |
| `last_synced_at` | date | ✓ | Date of last sync from the actual directory. |
| `doc_conventions` | object | ✓ | Document discovery and naming conventions. |
| `scope` | object | ✓ | Managed scope of versions / phases. |
| `roadmap_references` | list | — | Existing roadmap files (common in established projects). |
| `document_inventory` | list | ✓ | List of documents (external metadata). |
| `glossary` | list | — | Project-level standardized terminology; see `§Glossary`. |
| `code_conventions` | object | — | Code directory conventions (for sync use). |

---

## `doc_conventions` Sub-object

Describes "how user documents are organized." The skill reads these rules to discover and categorize documents.

```json
{
  "roots": ["Docs/AwesomeDoc/"],
  "version_grouping": {
    "enabled": true,
    "pattern": "AwesomeApp_Design_V(\\d+)",
    "example": "AwesomeApp_Design_V8"
  },
  "phase_grouping": {
    "enabled": true,
    "pattern": "Phase_([A-Z]+)",
    "examples": ["Phase_A", "Phase_B", "Phase_I"]
  },
  "design_doc_pattern": "**/*Phase*Design*Spec*.md",
  "roadmap_pattern": "**/*Roadmap*.md",
  "metadata_location": "external",
  "naming_template": "{project}_Design_V{version}-Phase_{phase}-{topic}_Spec.md"
}
```

### `metadata_location` Modes

| Mode | Meaning | Use Case |
|:---|:---|:---|
| `embedded` | Metadata written in the document header (YAML or Table). | Greenfield projects, newly created documents. |
| `external` | Metadata exists only in the `document_inventory[]` of this schema. | Brownfield projects; avoids intruding on existing documents. |
| `hybrid` | Embedded for new docs, External for legacy docs. | Gradual transition periods. |

**Default Recommendation**:
- `project_type=greenfield` → `embedded`.
- `project_type=brownfield` → `external`.

---

## `scope` Sub-object

```json
{
  "active_versions": ["V10"],
  "maintenance_versions": ["V8", "V9"],
  "archived_versions": ["V2", "V3", "V4", "V5", "V6", "V7"]
}
```

| Category | Stage 2 Optimization | Stage 3 Development | Stage 4 Sync |
|:---:|:---:|:---:|:---:|
| active | ✓ | ✓ | ✓ |
| maintenance | ✗ | Bug fixes only | ✓ |
| archived | ✗ | ✗ | ✗ |

For single-version projects, simply use `active_versions: ["main"]`.

---

## `roadmap_references` List

Used for brownfield projects to register "existing roadmap files" which are treated as `design-plan.md` in the skill's view.

```json
[
  {
    "file": "Docs/AwesomeDoc/AwesomeApp_Design_V10/Roadmap_v10.0.md",
    "version": "V10",
    "phase_table_section": "§2",
    "treated_as": "design_plan"
  }
]
```

When this field exists, Stage 1 will not force regeneration of `design-plan.md`; the skill will extract the phase table directly from the roadmap.

---

## `document_inventory` List (Core)

Each entry is equivalent to the **external metadata** of a design document.

```json
[
  {
    "doc_id": "awesome-app-v8-phase-i-session-lock",
    "path": "Docs/AwesomeDoc/AwesomeApp_Design_V8/Phase_I-SessionWriteLock_Spec.md",
    "version": "V8",
    "phase": "Phase I",
    "topic": "SessionWriteLock",
    "status": "implementing",
    "implementation_status": "partial",
    "score": { "total": 8.5, "accuracy": 9, "completeness": 8, "actionability": 9, "consistency": 8 },
    "owner": "Architect Group",
    "last_updated": "2026-05-04",
    "depends_on": ["awesome-app-v8-positioning"],
    "supersedes": [],
    "references": ["/path/to/peer-project"],
    "code_paths": [
      "crates/awesome-app/src/kernel/storage/session_write_lock.rs"
    ],
    "tags": ["phase-i", "storage"],
    "derived_from": "filename"
  }
]
```

### Mandatory Fields

`doc_id` / `path` / `version` / `phase` / `status` / `implementation_status`

### `derived_from` Enum

Indicates how the record was populated (for audit):
- `filename`: Regex-parsed from filename.
- `roadmap`: Parsed from a roadmap table.
- `manual`: Manually entered by user.
- `hybrid`: Merged from multiple sources.

---

## `code_conventions` Sub-object (Optional)

```json
{
  "language": "rust",
  "roots": ["crates/"],
  "workspace_file": "Cargo.toml",
  "test_command_template": "cargo test -p {crate} -- {test_name}",
  "ignore_patterns": ["**/target/**", "**/*.test.rs"]
}
```

Consumed by Stage 4 Sync for `file_exists` / `schema_match` checks.

---

## `glossary` List (Terminology)

The single source of truth for project-level terminology. Consumed by consistency checks in Stages 1, 2, and 4.

```json
[
  {
    "term": "Session",
    "definition": "Persistent state container for a single user session.",
    "synonyms": ["session object", "conversation"],
    "introduced_in": "awesome-app-v8-positioning",
    "stability": "stable"
  }
]
```

| Field | Description |
|:---|:---|
| `term` | Standard term (case-sensitive). |
| `definition` | One-sentence definition; revisions require bumping `stability` → `evolving` and registering a drift. |
| `synonyms` | Allowed synonyms (for soft pattern_check matching). |
| `introduced_in` | `doc_id` where this term was first introduced. |
| `stability` | `stable` / `evolving` / `deprecated`. |

---

## `document_inventory[].depends_on` Strengthening

Existing field preserved; strengthened into **graph constraints**:

- Non-empty `depends_on` forms a directed graph; schema writes **reject cycles** (verified before adoption/planning writes).
- Stage 1 Step 4 "Dependency Layering" must perform BFS on this graph; Layer 0 are root docs without parents.
- Stage 4 Sync warning `dependency_inversion` if a child doc is `approved` while parent is still `draft`.

---

## `project_constraints[]` (Top-level)

The authoritative source for defining boundaries between targets and references. All Stage 1-4 workflows MUST read and comply.

```json
"project_constraints": [
  {
    "id": "PC-001",
    "scope": "project",
    "category": "positioning",
    "statement": "AwesomeApp is an enterprise server project; positioned as Core Engine + API Service, NOT an IDE proxy.",
    "rationale": "Differentiated from industry-standard positioning.",
    "enforced_in": ["stage_1_planning", "stage_2_authoring", "stage_2_optimization", "stage_5_sync"]
  },
  {
    "id": "PC-002",
    "scope": "project",
    "category": "reference_usage",
    "statement": "Reference industry-standard/peer-system for concepts ONLY; do not port code.",
    "rationale": "Different tech stacks (e.g., Python vs Rust), different architectural positioning."
  },
  {
    "id": "PC-003",
    "scope": "project",
    "category": "scope_exclusion",
    "statement": "Do not replicate IDE-side Plugin System; handled by external agents."
  }
]
```

**Field Reference**:
- `category`: `positioning` / `reference_usage` / `scope_exclusion` / `tech_stack` / `compatibility` / `non_functional`
- `enforced_in[]`: Stages that MUST read and check this constraint (default: all)
- `id`: Stable ID for cross-referencing from inventory / inputs / drifts

**Consumers**:

| Consumer | Behavior |
|:---|:---|
| `01-planning.md §Step 0.1` | When writing `current_goal`, must resolve conflicts with `project_constraints` before committing |
| `prompts/reference-extraction.md` | "Violations of `category=reference_usage` → categorized as 'Not Adopted'" |
| `prompts/design-refine.md §Hard Constraints` | Docs must not violate any `project_constraints[].statement` |
| `03-design-optimization.md §5.3` injection block | Project constraints injected as sub-block in omc/omx prompts |
| `05-sync.md` | Adds `constraint_violation` soft-warning type |

---

## Constraints

1. **No Conflict with Embedded Metadata**: If a doc has both headers and inventory records, `metadata_location` determines the authority.
2. **No Silent Field Discarding**: Migrate, don't delete, old fields when upgrading `schema_version`.
3. **No Manual Deletion of Inventory Entries**: Mark as `status: superseded` instead to preserve audit trail.
4. **No Scanning of Archived Versions**: Stage 4 sync skips these to avoid scope explosion.
5. **All Writes MUST be CAS**: Read-compare-swap `revision`; no direct overwrites. Escalate after 3 conflict attempts.
6. **Reject Cycles**: `depends_on` graph must be checked for cycles before writing.
7. **Glossary is Additive Only**: Revisions to existing definitions must be done via new terms with `supersedes`, keeping old entries as `deprecated`.
