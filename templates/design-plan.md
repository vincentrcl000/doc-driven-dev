<!--
  Purpose: Template for a Design Optimization Plan.
  Function: Provides a roadmap for document creation and optimization, including quality scoring, dependency layering, and prioritization.
  Last Modified: 2026-05-06 12:00:00 (CST)
-->
# [Project] [Version] Design Document System Optimization Plan

| Field | Value |
|:---|:---|
| plan_id | `[project]-[version]-design-plan` |
| version | `0.1` |
| owner | `[Architect Group]` |
| last_updated | `YYYY-MM-DD` |
| baseline_doc | `[doc_id of the document used as a reference but NOT modified]` |
| target_quality | `"Independent implementation by weak models possible"` |

---

## 1. Overview and Quality Score Table

### 1.1 Scoring Dimensions (Dual Systems)

The skill maintains two sets of scoring systems, produced by different processes and written in parallel. Any field not yet evaluated remains as `—`.

#### 1.1.A 4-Dimensional Quality Score (Baseline, produced by `prompts/design-refine.md` / Stage 1 Initial Scan)

| Dimension | Description |
|:---|:---|
| **Accuracy** | Consistency between document content and current code implementation; no outdated descriptions. |
| **Completeness** | Coverage of key design decisions, boundary conditions, and parameter constraints. |
| **Actionability** | Capability for a weak model to implement directly from the doc; code examples are runnable. |
| **Consistency** | Alignment with baseline docs and other related documents. |

`quality.total` = Mean of the four dimensions (Scale: 10). Can be written by any process.

#### 1.1.B 6-Dimensional Deep Review Score (Produced by `prompts/multi-model-review.md §Step 6`)

| Dimension | Description |
|:---|:---|
| **Gap Alignment** | Coverage of the gap matrix against reference projects/roadmaps. |
| **Solution Completeness**| Presence of implementation strategy, edge cases, integration points, and error handling. |
| **Env Rigor** | Alignment with actual code stack, module boundaries, and constraints of the project. |
| **In-design Consistency** | Alignment of terminology/interfaces with `baseline_doc` and concurrent documents. |
| **Benchmark Rationality**| Appropriateness of adoption decisions (✅/🟡/🔄/❌) for reference projects. |
| **Anti-over-engineering**| Avoidance of introducing capabilities exceeding current needs. |

`deep_review.total` = Mean of the six dimensions (Scale: 10). Produced ONLY by Stage 3 Deep Mode.

### 1.2 Full Document Quality Score Table

| Doc | Name | Current Status | quality.total (4D) | deep_review.total (6D) | Last Scored | Priority |
|:---|:---|:---|:---:|:---:|:---:|:---:|
| `[doc-id]` | [Name] | [Status] | **[Mean]** | **[Mean]** | [Date] | [High/Mid/Low] |

> Dual-column coexist: `quality` is written by light/baseline processes; `deep_review` is written by Stage 3 Deep Mode. Un-evaluated fields remain `—`. Detailed sub-dimensions can be expanded in §1.2.A / §1.2.B (optional).

### 1.3 Classification of Core Issues

**Category A: Outdated Content / Contradiction with Implementation (MUST Fix)**
- [Doc]: [Specific issue]

**Category B: Missing Key Design (Impacts Actionability)**
- [Doc]: [Specific issue]

**Category C: Alignment Mismatch (Positioning/Terminology - Partial Fix)**
- [Doc]: [Specific issue]

**Category D: Complete but needs Supplement (Nice-to-have)**
- [Doc]: [Specific issue]

---

## 2. Dependencies and Optimization Order

### 2.1 Document Dependency Layer Map

```
【Layer 0 — Baseline (No Modification)】
  [Baseline Doc]

【Layer 1 — Foundation Layer (Priority Fix)】
  [Doc 1]       ← Referenced by [Doc A/B/C]
  [Doc 2]       ← ...

【Layer 2 — Core Specification Layer】
  ...

【Layer 3 — Integration & Operations Layer】
  ...

【Layer 4 — Frontend / Toolchain Layer】
  ...

【Layer 5 — Future Planning Layer (Macro Only)】
  ...
```

### 2.2 Recommended Execution Order

#### Batch 1: Foundation Layer

| Order | Doc | Core Tasks | Est. Effort |
|:---:|:---|:---|:---:|
| 1 | `[doc]` | [Task content] | Small/Mid/Large |

#### Batch 2: Core Specification Layer
<!-- Same as above -->

#### Batch 3: Integration & Operations Layer
<!-- Same as above -->

#### Batch 4: Frontend / Toolchain Layer
<!-- Same as above -->

#### Batch 5: Future Planning Layer
<!-- Same as above -->

### 2.3 Optimization Principles

1. **No Change to Positioning**: All modifications must align with `baseline_doc [doc-id]`. Baseline takes precedence in case of conflict.
2. **Verifiable Code References**: Code paths, function names, and line numbers must match the current codebase.
3. **Distinguish Current vs. Planned**: Use ✅ for implemented, ⏳/🔎 for planned.
4. **Abstract Macro Designs**: Layer 5 only requires architectural direction and interface boundaries.
5. **One Doc at a Time**: Avoid cross-doc sync confusion; update this plan after each completion.
6. **Reference Projects as Conceptual Benchmarks**: Do NOT describe as "Implementation taken from reference project."

### 2.4 Reference Sources

| Source | Path / URL | Benchmarking Target |
|:---|:---|:---|
| [Reference Project A] | `/path/to/your-project` | [Target capability] |

> This table is persisted in `{METADATA_DIR}/config.json` and auto-reused in Stages 2/3.

---

## 3. Detailed Guidance per Batch

### 3.1 Batch 1 — [Batch Name]

#### 3.1.1 `[doc-id]` — [Doc Name]

**Current Issues**

| Issue | Symptoms |
|:---|:---|
| [Issue] | [Symptoms] |

**Optimization Direction**

1. [Action 1]
2. [Action 2]

**Reusable Code Path References**

```
[Implementation File Path]: [Description]
```

---

## 4. Progress Tracking

| Doc | Assigned Batch | Responsible Model | Status | Completion Date | Remarks |
|:---|:---:|:---|:---:|:---|:---|
| `[doc-id]` | Batch 1 | [model] | [status] | [date] | [remarks] |

Status enum: `not_started`, `in_progress`, `in_review`, `done`.

---

## 5. Execution Log

### YYYY-MM-DD
- [Completed] `[doc-id]` finished first optimization round; score increased from X.X to Y.Y.
- [Decision] Introduced reference project `[name]` as benchmark for `[capability]`.

---

## 6. Driving Subsequent Stages

Once completed, this file directly drives Stage 2/3/4 of `doc-driven-dev`:

- Stage 2 Authoring: Call `workflows/02-design-authoring.md` doc-by-doc per §2.2.
- Stage 3 Optimization: Call `workflows/03-design-optimization.md` for existing docs using sources from §2.4.
- Stage 4 Sync: Auto-triggers upon completion of `metadata.code_paths` in docs.
