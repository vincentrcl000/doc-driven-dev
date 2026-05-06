<!--
  Purpose: General Template for a Design Document.
  Usage: Copy this file → Replace all placeholder brackets → Delete this HTML comment block.
  Function: Provides a standardized structure for design specifications, including metadata, architecture, and implementation details.
  Last Modified: 2026-05-06 11:55:00 (CST)
-->
# [Project] [Version] - [Phase] - [Topic] Design Specification

| Field | Value |
|:---|:---|
| doc_id | `[project]-[version]-[phase]-[topic]` |
| version | `0.1` |
| status | `draft` |
| owner | `[Responsible Person]` |
| last_updated | `YYYY-MM-DD` |
| depends_on | `[List of dependent doc_ids, or —]` |
| supersedes | `[Old doc_id being replaced, or —]` |
| phase | `[Phase X]` |
| implementation_status | `not_started` |
| references | `[Reference projects/docs, noting "Conceptual Benchmark"]` |
| code_paths | `[Corresponding code paths, or — (undetermined)]` |

---

## 0. Implementation Status

> This section provides readers with a quick way to locate information without reading the whole text.

### 0.1 Slice Planning

| Slice | Status | Description |
|:---|:---:|:---|
| X-1 [Core Skeleton] | [Status] | [One-line description] |
| X-2 [Main Features] | [Status] | [One-line description] |
| X-3 [Edge Cases] | [Status] | [One-line description] |

### 0.2 Reference Implementation

> The following references are for conceptual benchmarking only and do not involve code porting.

| Reference | Purpose | Adoption Level |
|:---|:---|:---|
| [Project/Lib] | [Capability targeted] | [Direct Adoption / Conceptual / Planned] |

### 0.3 Key Design Changes (Discrepancy Records)

| Change Point | Original Design | New Design | Rationale |
|:---|:---|:---|:---|
| [Item 1] | [Original] | [New] | [Rationale] |

### 0.4 Verification Log

- [ ] [Verifiable criterion with command or file path]
- [x] [Verified item]

### 0.5 Current Implementation Boundaries

- **Implemented**: [One-line description]
- **Not Implemented**: [One-line description]
- **Design Corrections**: [One-line description, if any]

### 0.6 Implementation Redlines

- **[Phase] Redline**: [Mandatory constraints to follow]
- **Postponement Redline**: [Capabilities NOT to be implemented in current phase]

---

## 1. Background & Mission

### 1.1 Current Problems

| Problem | Symptoms |
|:---|:---|
| [Problem 1] | [Specific symptoms] |

### 1.2 Core Mission

The objectives for [Phase] are:

1. **[Objective 1]**: [Brief description]
2. **[Objective 2]**: [Brief description]

---

## 2. Layered Architecture

### 2.1 Overall Structure

```mermaid
%% Or ASCII Diagram
```

### 2.2 Component Responsibilities

#### Implemented ([Slice ID])

| Component | Actual Path | Responsibility |
|:---|:---|:---|
| [Component] | `[path/to/file.ext]` | [Responsibility] |

#### Planned ([Target Slice ID])

| Component | Target Slice | Responsibility |
|:---|:---|:---|
| [Component] | [Slice] | [Responsibility] |

---

## 3. Core Component Design

> Note: This section focuses on "Design Decision Summaries" and avoids large blocks of pseudocode. Use decision tables to explain key choices.

### 3.1 [Component A] — [One-line description]

#### 3.1.1 Current Implementation ([Slice ID])

**Key Design Decisions**:

| Decision | Choice | Rationale |
|:---|:---|:---|
| [Decision Point] | [Choice] | [Why] |

**Implemented Behavioral Rules** (By priority):

| Rule | Behavior | Trigger Scenario |
|:---|:---|:---|
| [Rule 1] | [Action] | [When] |

#### 3.1.2 Future Planning ([Target Slice ID])

[Briefly describe subsequent slice enhancements; no code]

### 3.2 [Component B] — [...]
<!-- Same structure as above -->

---

## 4. Implementation Phases (Completed / Planned)

### Phase X-1: [Slice Name] [Status Marker]

**Goal**: [One-line]

**Accomplished**:
- [Completed item]

**Not Done**:
- [Incomplete item]

### Phase X-2: [Slice Name] [Status Marker]
<!-- Same structure as above -->

---

## 5. Key Tech Details

### 5.1 [Tech Point 1]

[3-5 lines of decision summary; code snippets ≤ 10 lines if necessary]

### 5.2 [Tech Point 2]
<!-- ... -->

---

## 6. Key Risks & Countermeasures

### 6.1 [Risk 1] [Status Marker (✅/⚠️/🔜)] ([Slice ID])

- **Risk**: [One-sentence trigger scenario]
- **Countermeasure**: [Mitigation action]
- **Residual Risk**: [If any]

### 6.2 [Risk 2] [...]
<!-- ... -->

---

## 7. Acceptance Criteria

### 7.1 [Completed Slice] Acceptance ✅

1. [x] [Verifiable criterion with command]
2. [x] ...

### 7.2 [Target Slice] Acceptance Criteria 📋

1. [ ] [Verifiable criterion]
2. [ ] ...

---

## 8. File Planning

### 8.1 Created ([Slice ID])

| File | Status | Description |
|:---|:---:|:---|
| `[path/to/file.ext]` | ✅ | [Description] |

### 8.2 Planned New ([Target Slice ID])

| File | Target Slice | Description |
|:---|:---|:---|
| `[path/to/file.ext]` | [Slice] | [Description] |

### 8.3 Planned Modifications ([Target Slice ID])

| File | Target Slice | Change |
|:---|:---|:---|
| `[path/to/file.ext]` | [Slice] | [What to change] |

### 8.4 Modules NOT to be Introduced (Discrepancy)

| Original Design | Actual Decision | Rationale |
|:---|:---|:---|
| [Original Module] | [Not created] | [Rationale] |
