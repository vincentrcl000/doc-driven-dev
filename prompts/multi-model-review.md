<!--
  Purpose: Prompt template for Multi-model Review 9-Step Process.
  Function: Orchestrates multiple LLMs to iteratively draft, review, correct, and finalize design documents until convergence.
  Last Modified: 2026-05-06 11:40:00 (CST)
-->
# Prompt: Multi-Model Review 9-Step Process

> Purpose: Use multiple LLMs to iteratively draft, review, correct, and finalize design documents until they converge.

## Step 0 — Capability Probing & Routing (Mandatory Pre-condition)

**Must be completed before entering this process.** Otherwise, the process degrades into a single model talking to itself, losing the value of cross-review.

### 0.1 Read Available Model List

Priority order:

1. Read `{METADATA_DIR}/config.json.models[]` (Written by adoption or this Step 0). Structure:
   ```json
   "models": [
     { "id": "claude-opus-4", "role_pref": "drafter",  "available": true,  "switch_hint": "/model claude-opus-4" },
     { "id": "claude-sonnet-4",  "role_pref": "reviewer", "available": true,  "switch_hint": "/model claude-sonnet-4" },
     { "id": "gpt-5",            "role_pref": "judge",    "available": false, "switch_hint": "codex --model gpt-5" }
   ]
   ```
2. If `models[]` is missing or empty → `AskUserQuestion` to let user list "Available model IDs + Switch commands for the current host," and write back to config (idempotent append).
3. If the user says "only one" or nothing is configured → Enter §0.4 Single-Model Downgrade.

### 0.2 Routing Level Determination

| Condition | Level | Description |
|:---|:---|:---|
| ≥ 3 distinct models available | **L3 — True Multi-model** | Assign separate models for Drafter, Reviewer, and Judge. |
| 2 distinct models available | **L2 — Dual-model** | Use different models for Draft vs. Review; Judge reuses the stronger reasoning model in a separate session. |
| 1 model available | **L1 — Single-model Multi-persona** | Follow §0.4 downgrade strategy; relax convergence criteria. |
| 0 models (Anomaly) | Abort process and inform user. | — |

The determination result is written to `state.json.review_session.level` and used throughout the 9 steps.

### 0.3 Explicit Switch Protocol (L2 / L3)

Each time a new Step is entered requiring a role change, **the switch command must be output for the user to execute**, waiting for confirmation before continuing:

| Host | Switch Method (Example) |
|:---|:---|
| Claude Code | `/model <model-id>` then start a new conversation (`/new` or UI). |
| Codex | `codex --model <model-id>` to restart session, or switch to another profile. |
| Windsurf / Cursor / IDE | Manual switch in UI + Start new session. |

The skill output must be formatted like:

```
🔁 Entering Step 4 (Model 2 Independent Draft)
   This step requires switching to the Reviewer model: claude-sonnet-4
   Please execute: /model claude-sonnet-4 && /new
   Reply "ready" when done; I will provide the context as a standalone session.
```

**Independence Guarantee**: In the new session, **paste only the minimum context required for Step 4** (Target section + Reference highlights + Constraints); **do NOT paste the draft already written by Model 1** (unless explicitly required by the step, e.g., Step 5 Merge).

### 0.4 Single-Model Downgrade (L1)

When only one model is available, simulate cross-review with "Multi-persona + Red-team Self-audit":

1. **Persona Injection**: Reset persona at the start of each Step using system-style prompts:
   ```
   You are now the [Reviewer]. Forget all previous judgments made as the [Drafter].
   Your KPI is to find flaws in the solution, not to agree with it.
   Your ONLY information source for the current project is the following material: <Paste only necessary materials>
   ```
2. **Session Independence Equivalent**: **Clear context or open a new session** before each persona switch (e.g., Claude Code `/clear`, Codex new session); skill must explicitly prompt user.
3. **Red-team Reinforcement**: Reviewer personas in Step 4 / 6 / 9 must include "Red-team instructions":
   - "Assume by default that the solution has at least 3 critical flaws; find them."
   - "If you find none, you haven't reviewed carefully enough; review again."
4. **Relaxed Convergence Criteria**:
   - Step 9 requires only **1 round** of ✅ (L3 requires 2 consecutive).
   - Score threshold for 4 dimensions adjusted from ≥ 8 to ≥ 7.
   - **Must** add a mandatory "Human Final Review" item.
5. **Loss Declaration**: Inform the user at the start: "Currently in L1 single-model downgrade; review independence is weaker than true multi-model; please increase human intervention during final review."

### 0.5 Write-back on Session Completion

Append an entry to `state.json.review_session[]` after completion:

```json
{
  "doc_id": "xxx-v10-phase-a-session",
  "level": "L2",
  "models_used": ["claude-opus-4", "claude-sonnet-4"],
  "rounds": 2,
  "converged": true,
  "timestamp": "..."
}
```

This serves to audit that "this document was indeed multi-model reviewed, not just self-praised."

---

## Role Assignment

| Role | Responsibility | Recommended Model Positioning |
|:---|:---|:---|
| **Model 1 (Drafter)** | Research references, produce initial draft, revise based on feedback. | Strong code reading, long context (e.g., GPT-4 / Claude Opus). |
| **Model 2 (Reviewer)** | Gap analysis against references, find flaws, suggest improvements. | Critical reasoning (e.g., Claude Sonnet / Gemini Pro). |
| **Model 3 (Judge)** | Evaluate optimality, identify over-engineering. | Architectural judgment (can be same model as Model 2 but in separate session). |

**In L1 Downgrade**: Three roles played by different personas of the same model + separate sessions, following §0.4 rules.

---

## Standard 9-Step Process

### Step 1 (Model 1): Research Reference Implementation

**Input Prompt Template**:

```
Reference Projects: [Path 1] [Path 2]
Target Topic: [Design doc topic + Corresponding section/capability]
Project Positioning Doc: [Path] (Clear boundaries of current project)

Please answer:
1. How is this capability implemented in the reference projects? (Core mechanism, key files, data flow)
2. What is "nice-to-have" vs. "core" in their implementation?
3. What are the pitfalls/traps?
4. What depends on reference-project-specific infrastructure?

Output Format: Decision table + Key code path references + Summary of adoption suggestions.
```

### Step 2 (Model 1): Drafting Initial Version

**Input Prompt Template**:

```
Based on Step 1 research, author the design doc for the current project:
- Benchmark Doc: [Path of an existing high-quality doc]
- Target Doc Path: [Path]
- Project Conventions: [Fill in actual values for convention template]
- Design Principles:
  1. Benchmark gap analyze, do NOT directly port code.
  2. Anti-over-engineering, prune to minimum viable for now.
  3. Status layering: Implemented / Partial / Planned.
  4. Use decision tables, avoid pseudocode > 10 lines.

Please write section-by-section, 1-2 chapters at a time, waiting for user confirmation.
```

### Step 3 (Model 1): Deepening the Draft

When user feedback says "Depth is insufficient," the drafter enriches:

```
Section depth requirements for Benchmark Doc [Path] (All outputs per 03-design-optimization.md §1.6 morphological constraints; NO implementation code):
- Implementation Strategy (Non-code): Decision table / Module map / Contract snippets ≤ 10 lines / Data flow diagrams.
- Edge Case List.
- Integration Point descriptions (Interactions with other components; module map + calling contract, no implementation).
- Output Handling Strategy.
- Execution Mode.
- Configuration Model.
- Implementation Slicing/Phases.
- Error Handling Matrix.

Redlines: Single pseudocode/code block > 10 lines fails; implementation blocks with function bodies fail; introducing crates/types/paths not yet in project is "creating code."

Continue supplementing section-by-section based on the previous draft.
```

### Step 4 (Model 2): Independent Comparison Solution

When Model 1's quality is questionable for a critical capability, let Model 2 draft independently:

```
[Standalone context for Model 2, Model 1's draft NOT provided]

Benchmark [Capability] from reference project [Path] combined with current project foundation:
[Project status summary + Relevant completed docs]

Please independently draft a design solution for [Topic]. Evaluation points:
1. Gap Analysis: What to adopt from reference vs. what not to.
2. Consistency with existing designs [List relevant docs].
3. Anti-over-engineering: Capabilities to postpone to later slices.
```

### Step 5 (Model 1): Compare & Merge / Verify User Rewrite

Covers two trigger scenarios:

#### Scenario A — Merging after Model 2's Independent Draft

```
Model 2's independent solution: [Paste]
My previous draft: [Paste]

Compare both solutions and produce a merged version:
- Adopt reasonable improvements from Model 2.
- Preserve strengths of my original draft.
- If essential conflicts exist, list rationales for user decision.
```

#### Scenario B — Verification after User Manual Rewrite

```
I have manually rewritten this draft: [Paste new draft]
Original Objective: [goal_kind / Benchmark / Reference from §1 inputs]

Check this draft for deviations in:
1. Alignment with target topic and project positioning.
2. Benchmarking of [reference_projects] (did it drift?).
3. Depth gap compared to [benchmark_docs].
4. Presence of all sections in §1 inputs.required_sections.

Output: Deviation list (sorted by severity) + Recommendation for Step 6 Final Review.
```

### Step 6 (Model 3): Holistic Evaluation

```
[Model 3 standalone session]

Evaluate the following design doc: [Path or Full text]

Evaluation Dimensions:
1. Gap Alignment: Compliance with [Gap Analysis Doc] coverage.
2. Solution Completeness: §0~§8 sections present and substantively filled.
3. Rigor in Current Environment: Referenced code paths actually exist.
4. Consistency with Existing Design: [List relevant docs].
5. Rationality of Adoption: Decision-making regarding [Reference Project] benchmarking.
6. Anti-over-engineering: Identified capabilities exceeding current needs.

Provide improvement list + Overall score (10-point scale, 4 dimensions).
```

### Step 7 (Model 2 or Model 1): Adoption Decision

For every suggestion from Model 3:

```
Suggestion N: [Content]

Please judge:
- Rationale for Adoption / Rejection.
- If adopted, categorize into A/B/C/D revision.
- Expected score improvement after revision.
```

Final go/no-go decision by the user.

### Step 8 (Model 3): Verification After Revision

```
Revision completed per suggestions; updated draft: [Paste]

Please check:
- Is every suggestion implemented?
- Are new issues introduced?
- Consistency with related docs maintained?

Output: ✅ Pass / ⚠️ Further revision required [specific items]
```

### Step 9 (Model 3): Final Convergence

```
Perform final review of the draft:

Evaluate against current project architecture, reference benchmarking, and anti-over-engineering:
1. Is it now the optimal solution? If not, what can still be improved?
2. Are there still uncovered risks / edge cases?
3. Is the implementation path clear and traceable?

If all pass → ✅ Converged, `status` can advance to `approved`.
If suggestions remain → Return to Step 7.
```

---

## Convergence Criteria

**L3 (≥ 3 Models)**: All must be met.
- [ ] Model 3 gives ✅ for **two consecutive rounds** of Step 9.
- [ ] All 12 items in `design-refine.md` checklist pass.
- [ ] Scores for all 4 dimensions ≥ 8.
- [ ] User confirmation.

**L2 (2 Models)**:
- [ ] Model 3 gives ✅ for **at least one round** of Step 9 + Model 2 independent review gives ✅.
- [ ] All checklist items pass + 4D scores ≥ 8.
- [ ] User confirmation.

**L1 (1 Model Downgrade)**:
- [ ] Reviewer persona gives ✅ under Red-team instructions.
- [ ] All checklist items pass + 4D scores ≥ 7.
- [ ] **Mandatory** human final review pass (No "AI Verified" substitution).

---

## Anti-Divergence Rules

- **Round Limit**: Step 1-9 for a single doc must not exceed 3 major cycles; exceeding suggests fundamental issues in the solution or unclear requirements; escalate to architecture meeting instead of continuing AI cycles.
- **Split Resolution**: If Model 1 and Model 2 diverge on a core decision after 3 rounds, it MUST be escalated for user decision; do not let models continue debating.
- **Reference Overload Protection**: No more than 3 reference projects in Step 1; exceeding this dilutes the comparison effect.
