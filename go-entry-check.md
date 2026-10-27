# 老项目入场检测 · Brownfield 必跑（GO 分片 · 第三步）

> 分片自 GO.md · 按需整读 · R1.10 体系。
> **何时整读我**：STATE 无 CONTEXT/技术栈行且 .specs 不存在时，进 0-change 前整读我执行 3.1–3.4；命中任一豁免（R1.10 冷启动例外）直接跳过。

## 第三步 · 老项目入场检测（brownfield 必跑）

**触发**：进入 0-change 之前必跑。

### 3.1 探测 AI 上下文文档

按下面顺序 `ls` / `find` 探测：

| 文档 | 路径 | 来自哪个生态 |
|---|---|---|
| `CONTEXT.md` | 仓库根 / `.specs/` | code-kit 自己 |
| `AGENTS.md` | 仓库根 | OpenAI Codex / 标准 agents 协议 |
| `CLAUDE.md` | 仓库根 / `.claude/` | Anthropic Claude Code |
| `.cursor/rules/*.md` | `.cursor/` | Cursor IDE |
| `.windsurf/rules/*.md` | `.windsurf/` | Windsurf IDE |
| `.github/copilot-instructions.md` | `.github/` | GitHub Copilot |
| `.clinerules` | 仓库根 | Cline |

读 `STATE.md` 的 `ai_context_doc`（用户上次指定的替代文档）和 `last_intel_scan` 字段。

### 3.2 判决

#### 情况 A · `STATE.md` 已设 `ai_context_doc`

用户上次明确指定了某文档为 AI 遵守依据。**直接读它**，不再询问。

后续阶段（2-design / 4-dev）改读 `<ai_context_doc>` 替代 `.specs/CONTEXT.md`。

#### 情况 B · 已存在 CONTEXT.md 且 `last_intel_scan` 在 90 天内

**直接读 CONTEXT.md**，跳过本步。无需打扰用户。

#### 情况 C · 已存在 CONTEXT.md 但超过 90 天

读 CONTEXT.md，**提醒用户**："上次扫描已 X 天，可重跑 intel-scan"，但**不强制**。

#### 情况 D · 未发现 CONTEXT.md，但有其他 AI 上下文文档（AGENTS / CLAUDE / Cursor / 等）

**反问用户**（**必须**等待回复才能继续）：

```
🔍 检测到本项目已有以下 AI 上下文文档：
  - <列出找到的，含路径与文件大小>

code-kit 默认用 CONTEXT.md 作为单一源。请选择：
  1. 跑 intel-scan，综合现有文档 + 代码扫描，生成 CONTEXT.md（推荐）
  2. 以现有文档为准（告诉我哪个），跳过 intel-scan
  3. 跳过 intel-scan + 不读现有文档（不推荐 · AI 会盲飞）

请选 1/2/3。无人工确认前我不进 0-change（避免 AI 在不知项目约定下开始动手）。
```

- **选 1**：进 `prompts/I-intel-scan.md` 走分支 A
- **选 2**：在 `STATE.md` 写 `ai_context_doc: <用户指定路径>`，回到原意图（直接进 0-change）
- **选 3**：在 `STATE.md` 写 `ai_context_doc: none / skip`，回到原意图（带警告）

#### 情况 E · 未发现任何 AI 上下文文档（CONTEXT / AGENTS / CLAUDE / Cursor / Windsurf / Copilot / Cline 全无）

**反问用户**（**必须**等待回复才能继续）：

```
🔍 项目里未发现任何 AI 上下文文档。

code-kit 后续阶段需要项目上下文给 AI 用。请选择：
  1. 现在跑入场扫描，自动生成 CONTEXT.md（~15-30k tokens · 仅首次 · 推荐）
  2. 我手动指定项目里某个文档作为开发遵守依据：<请回复路径>
  3. 跳过 intel-scan，直接进 0-change（不推荐 · AI 会"盲飞"，老项目护栏 B1-B5 全失效）

请选 1/2/3。无人工确认前我不开始扫描或进入 0-change（避免无意义消耗 token）。
```

- **选 1**：进 `prompts/I-intel-scan.md` 走分支 C
- **选 2**：在 `STATE.md` 写 `ai_context_doc: <路径>`，回到原意图
- **选 3**：在 `STATE.md` 写 `ai_context_doc: none`，回到原意图（带警告）

#### 情况 F · 是刚创新项目（无 `package.json` / `pyproject.toml` / 等代码上下文）

跳过本步。这是 greenfield 项目，CONTEXT.md 会在 0-change / 1-requirement / 2-design 过程中逐步沉淀。

### 3.3 为什么这步重要

跳过会导致：
- AI 不知项目架构 → 写出不合项目风格的代码
- AI 重复实现已有抽象 → 费 token 且产生重复代码
- `4-dev-schema.md` schema 任务 / `4-dev-breaking.md` 破坏性变更检测都会不准
- 用户明明有 CLAUDE.md 写好的约定 → AI 完全不读 → 抱怨"为什么不按我说的来"
