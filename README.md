# doc-driven-dev

<p align="center">
  <a href="#english">English</a> | <a href="#chinese">简体中文</a>
</p>

---

<a name="english"></a>
# English Version

> **Closed-Loop Document-Driven Development (DDD) Orchestrator** — A complete lifecycle orchestrator from "Design Planning" to "Doc-Code Consistency Sync."

## Core Features

- **Schema-Driven, Zero-Intrusion**: Supports rapid adoption for existing (Brownfield) projects. No need to modify any existing document content; the skill understands the mapping between documents and code through an external project profile.
- **Native Support for Claude Code & Codex**:
    - **Host Awareness**: Automatically identifies the environment and configures metadata storage paths (`.claude/` or `.codex/`).
    - **Automated Batching**: When performing large-scale document optimization or authoring, it automatically invokes **OMC (oh-my-claudecode)** or **OMX (oh-my-codex)** for multi-threaded/asynchronous orchestration, significantly boosting efficiency.
- **Greenfield / Brownfield Dual Paths**: Provides different takeover strategies for new and old projects. New projects get standard skeletons, while old projects are indexed via read-only scanning.
- **Built-in Drift Detection**: Automatically verifies if the code implementation leads the documentation. When the code adds capabilities not described in the docs, a warning is triggered to drive plan revision.
- **Three-File Responsibility Layering**:
    - `project-schema.json`: Project fingerprint (Doc inventory, version grouping, mapping rules).
    - `config.json`: Skill behavior (Reference sources, sync rules, host preferences).
    - `state.json`: Dynamic state (Current task, known drifts, session logs).

## Prerequisites

This skill is designed for the **Oh My Agent** ecosystem. To enable slash commands (`/`) and automatic recognition, you **must** have the corresponding orchestrator installed:
- **For Claude Code**: Install [OMC (oh-my-claudecode)](https://github.com/vincentrcl000/oh-my-claudecode).
- **For Codex**: Install [OMX (oh-my-codex)](https://github.com/vincentrcl000/oh-my-codex).

---

## Installation & Setup

### For Claude Code
1. Copy the `doc-driven-dev` directory to `~/.claude/skills/` on your machine.
2. Type `Enable doc-driven-dev in this project` to initialize.

### For Codex
1. Copy the `doc-driven-dev` directory to `~/.codex/skills/` on your machine.
2. Run `omx reload skills` in your terminal or restart your Codex session.
3. Type `/skill doc-driven-dev` (or `/` to select from the list) to activate.

## Usage

### Mode A: Natural Language (Recommended)
Explicitly mention the skill name when initializing or executing cross-stage tasks to ensure the AI invokes the correct workflow.
- **Project Adoption**: "Use **doc-driven-dev** to manage this project's document system," "Enable **doc-driven-dev**."
- **Doc Optimization**: "Use **doc-driven-dev** to optimize this design spec and align it with industry-standard implementation," "Batch run **doc-driven-dev** to optimize all V10 Phase A documents."
- **Implementation**: "Implement the core write-lock logic according to the **doc-driven-dev** design spec," "Sync **doc-driven-dev** development plan progress."
- **Consistency Sync**: "Run **doc-driven-dev** to check for code-doc drift," "View the **doc-driven-dev** current progress dashboard."

### Mode B: Explicit Commands (CLI Style)
Pass arguments in the chat box to skip guidance and enter a specific Stage.
- `doc-driven-dev plan`: Create or update design/development plans.
- `doc-driven-dev design --reference /path/to/ref`: Author or optimize design based on references.
- `doc-driven-dev dev --stage A`: Manage development implementation for a specific phase.
- `doc-driven-dev sync`: Perform a full doc-code consistency check.

## Typical Workflows

### Greenfield (New Project)
```
User: "Enable doc-driven-dev in this new project"
 → Stage 0 Adoption: Interactive collection → Create docs/ skeleton + project-schema
 → Stage 1 Plan: Generate design and development plan dashboards
 → Stage 2 Design: Section-by-section drafting + multi-model optimization
 → Stage 3 Develop: Code implementation traceable to design + incremental sync
 → Stage 4 Sync: Periodic full check to close the loop on all drifts
```

### Brownfield (Existing Project)
```
User: "Apply doc-driven-dev to AwesomeApp"
 → Stage 0 Adoption: Read-only scan of existing docs → Identify versions/phases → Build external inventory
 → Mode Selection:
    ├─ Initial Health Check: Run full sync to expose existing inconsistencies
    ├─ Targeted Optimization: Deep refine and de-pseudocode a single stale doc
    └─ Systemic Revision: Formulate a global sync plan based on the existing roadmap
```

---

<a name="chinese"></a>
# 中文版

> **文档驱动开发 (DDD) 闭环技能** — 从"设计计划"到"文档-代码同步"的完整生命周期编排。

## 核心特性

- **Schema-driven，零侵入**：支持存量项目（Brownfield）快速接入。无需修改任何现有文档内容，技能通过外部项目画像理解文档与代码的映射关系。
- **Claude Code & Codex 原生支持**：
    - **宿主感知**：自动识别环境并配置元数据存储路径（`.claude/` 或 `.codex/`）。
    - **自动化批处理**：在执行大规模文档优化或编写任务时，会自动调用 **OMC (oh-my-claudecode)** 或 **OMX (oh-my-codex)** 进行多线程/异步编排，大幅提升效率。
- **Greenfield / Brownfield 双路径**：针对新老项目提供不同的接管策略。新项目自动生成标准骨架，老项目只读扫描并建立索引。
- **内置漂移检测 (Drift Detection)**：自动核查代码实现是否领先于文档。当代码新增了文档未描述的能力时，自动触发预警并驱动计划修订。
- **三文件职责分层**：
    - `project-schema.json`：项目画像（文档清单、版本分组、映射规则）。
    - `config.json`：技能行为（参考来源、同步规则、宿主偏好）。
    - `state.json`：动态状态（当前任务、已知 Drift、会话日志）。

## 前提条件

本技能专为 **Oh My Agent** 生态设计。为了启用斜杠命令 (`/`) 和自动识别功能，你**必须**安装对应的编排器：
- **对于 Claude Code**：安装 [OMC (oh-my-claudecode)](https://github.com/vincentrcl000/oh-my-claudecode)。
- **对于 Codex**：安装 [OMX (oh-my-codex)](https://github.com/vincentrcl000/oh-my-codex)。

---

## 安装与配置

### 在 Claude Code 中安装
1. 将 `doc-driven-dev` 目录复制到你机器的 `~/.claude/skills/` 下。
2. 输入 `在这个项目启用 doc-driven-dev` 即可完成初始化。

### 在 Codex 中安装
1. 将 `doc-driven-dev` 目录复制到你机器的 `~/.codex/skills/` 下。
2. 在终端运行 `omx reload skills` 或重启 Codex 会话以刷新技能列表。
3. 输入 `/skill doc-driven-dev`（或输入 `/` 从列表中选择）即可激活。

## 使用方式

### 方式 A：自然语言描述（推荐）
在初次启用或执行跨阶段任务时，建议显式提及技能名称以确保 AI 准确调起对应工作流。
- **接管项目**：“使用 **doc-driven-dev** 帮我把这个项目的文档体系管起来”、“启用 **doc-driven-dev**”。
- **优化文档**：“用 **doc-driven-dev** 优化这份设计稿，对齐下 industry-standard 的实现”、“批量运行 **doc-driven-dev** 优化 V10 阶段 A 的所有文档”。
- **按图施工**：“按 **doc-driven-dev** 的设计方案实现核心写锁逻辑”、“同步 **doc-driven-dev** 的开发计划进度”。
- **一致性核查**：“跑一下 **doc-driven-dev** 检查代码和文档的漂移”、“查看 **doc-driven-dev** 当前进度总表”。

### 方式 B：显式指令（命令行风格）
在对话框中输入参数可直接跳过引导，进入特定 Stage。
- `doc-driven-dev plan`：制定或更新设计/开发计划。
- `doc-driven-dev design --reference /path/to/ref`：基于参考项目编写/优化设计。
- `doc-driven-dev dev --stage A`：管理特定阶段的开发实施。
- `doc-driven-dev sync`：执行全量文档-代码一致性检查。

## 典型使用链路

### 新项目 (Greenfield)
```
用户："在这个新项目启用 doc-driven-dev"
 → Stage 0 Adoption：交互收集需求 → 创建 docs/ 骨架 + project-schema
 → Stage 1 Plan：生成设计与开发计划总表
 → Stage 2 Design：逐章节编写设计方案 + 自动多模型优化
 → Stage 3 Develop：基于设计方案编写代码 + 任务级增量核查
 → Stage 4 Sync：周期性全量核查，闭环修复所有 Drift
```

### 已有项目 (Brownfield)
```
用户："帮我把 doc-driven-dev 用到 AwesomeApp"
 → Stage 0 Adoption：只读扫描存量文档 → 识别版本与阶段 → 建立外部索引 (Inventory)
 → 模式选择：
    ├─ 先体检：运行全量核查，暴露存量代码与文档的不一致
    ├─ 局部优化：针对单份陈旧文档进行深度打磨与去伪代码化
    └─ 体系修订：基于现有路线图，制定全局同步计划
```
