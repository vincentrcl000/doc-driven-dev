<!--
  Purpose: Specification for Document Metadata (Embedded Mode).
  Function: Defines the schema for the "metadata" block used in design documents (embedded or hybrid modes).
  Last Modified: 2026-05-06 12:10:00 (CST)
-->
# Specification: Document Metadata Schema (Embedded Mode)

> **Scope**: This specification applies ONLY when `project-schema.metadata_location` is set to `embedded` or `hybrid` — i.e., for **newly created** design documents or full usage in Greenfield projects.
>
> **Brownfield existing documents do NOT need modification per this spec.** For legacy documents where `metadata_location=external`, all fields described here exist within the `document_inventory[]` entries of `{METADATA_DIR}/project-schema.json` (sharing the same names and semantics) as defined by `schema/project-schema.md`, and are **NOT written into the documents themselves**.

## Mandatory Fields

| Field | Type | Description |
|:---|:---|:---|
| `doc_id` | string | Unique document ID, recommended format: `{project}-{version}-{phase}-{topic}`. |
| `version` | string | Document version (decoupled from project version); bump on revision. |
| `status` | enum | `draft` / `in_review` / `approved` / `implementing` / `superseded`. |
| `owner` | string | Primary responsible person / role. |
| `last_updated` | date | ISO date, e.g., `2026-05-05`. |
| `depends_on` | list | List of `doc_id`s for prerequisite design documents; nullable. |
| `supersedes` | list | `doc_id`s of old documents replaced by this one; nullable. |
| `phase` | string | Phase identifier (e.g., `Phase I`, `Phase B`). |
| `implementation_status` | enum | `not_started` / `partial` / `implemented` / `deprecated`. |

## Optional Fields

| Field | Type | Description |
|:---|:---|:---|
| `references` | list | Reference sources (Project paths, URLs, papers). |
| `code_paths` | list | Corresponding code paths (Consumed directly by Stage 4 Sync). |
| `reviewers` | list | List of reviewers. |
| `review_count` | int | Number of completed multi-model review rounds. |
| `score` | object | Score: `{ accuracy, completeness, actionability, consistency, total }`. |
| `tags` | list | Searchable tags. |
| `template_version`| string | Version of `templates/design-doc.md` used for creation/migration (e.g., `1.1`). |
| `last_reviewed_with`| object| Prompt version fingerprints used in last review: `{ "design_refine": "1.0", "multi_model_review": "1.0" }`. |
| `review_session_id`| string | Backlink to `state.json.review_session[].id` for the latest multi-model review. |

## Recommended Format (Table - Human Friendly)

```markdown
| Field | Value |
|:---|:---|
| doc_id | awesome-app-v8-phase-i-session-lock |
| version | 0.3 |
| status | approved |
| owner | Architect Group |
| last_updated | 2026-05-05 |
| depends_on | awesome-app-v8-positioning |
| supersedes | — |
| phase | Phase I |
| implementation_status | partial |
| references | IndustryStandard session-write-lock.ts (Concept Benchmark) |
| code_paths | crates/awesome-app/src/kernel/storage/session_write_lock.rs |
| score | 8.5 / 10 (accuracy 9, completeness 8, actionability 9, consistency 8) |
```

## Alternative Format (YAML Frontmatter - Machine Friendly)

```yaml
---
doc_id: awesome-app-v8-phase-i-session-lock
version: 0.3
status: approved
owner: Architect Group
last_updated: 2026-05-05
depends_on: [awesome-app-v8-positioning]
supersedes: []
phase: Phase I
implementation_status: partial
references:
  - type: concept-reference
    name: IndustryStandard session-write-lock.ts
code_paths:
  - crates/awesome-app/src/kernel/storage/session_write_lock.rs
score:
  accuracy: 9
  completeness: 8
  actionability: 9
  consistency: 8
  total: 8.5
---
```

## Usage Rules

1. **Initial Creation**: `status=draft`; requires `doc_id`, `owner`, `last_updated`, and `phase`.
2. **Under Review**: `status=in_review`; must supplement `depends_on` and `references`.
3. **Finalized**: `status=approved`; must include full `score`.
4. **Implementing**: `status=implementing` + `implementation_status=partial`; `code_paths` must be provided.
5. **Deprecated**: `status=superseded`; specify the superseding document at the top of the body.

## Alignment with Stage 4 Sync

The `code_paths` field (whether in header or inventory) is the ONLY source of mapping for Stage 4 `workflows/05-sync.md`.

| metadata_location | Authoritative Source | Discrepancy Handling |
|:---|:---|:---|
| `embedded` | Header Metadata (inventory as cache). | Header/Inventory mismatch → Drift. |
| `external` | `document_inventory` | Any metadata statements in the doc are ignored. |
| `hybrid` | Header preferred; Fallback to inventory. | Union of code_paths from both locations is checked. |

`implementation_status=implemented` with missing `code_paths` files will be directly identified as P1 drift, regardless of the authority source.
