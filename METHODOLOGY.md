# 方法论（code-kit 骨架 · AI 必读）

> **你是 AI 助手。这是 code-kit 的方法论全文，每次会话开始 / 清窗恢复时必读一遍。**
> 读完之后你应该知道：当前处于哪个阶段、要产出什么工件、和其他阶段的输入输出关系、哪些核心机制不允许绕过。
> 后续具体执行按各阶段 `prompts/<n>-*.md` 的指令走，本文件只定义骨架。

---

## BMAD 融合层

BMAD 能力通过 `BMAD-INTEGRATION.md` 与 `prompts/B-bmad-extensions.md` 接入本骨架，不另建一套产物体系：产品能力补到 CHANGE/REQUIREMENT，架构能力补到 ARCHITECTURE/DESIGN，Epic/Story/Sprint 补到 TASK/STATE，QA/复盘补到 TEST/REVIEW/INTEGRATION。重大变更走 `prompts/C-course-correction.md`。

**大议题/子议题例外锁**：`prompts/D-discovery.md` 保留 BMAD 的完整流程——大议题每轮八步、最多三轮；第 7 步发现子议题、第 8 步人工确认；确认后才逐个讨论子议题，子议题结束也必须人工确认。任何未确认结论不得落盘。不得将子议题自动扩展为新的三轮大议题。

---

```
[DISCOVERY]* → CHANGE → REQUIREMENT → DESIGN → [2a UI-DESIGN]* → TASK → DEV → TEST → REVIEW → [S-ALIGN]** → INTEGRATION → ARCHIVE
     │             │          │           │            │            │      │       │         │            │            │
     │             │          └─ CONTEXT.md 跨阶段共享 ┘     前端项目 ┘     └ TDD ─┘         │            │            │
     │             │                                                                      │            │            │
     │             └─────────────────────────── 迭代回灌（new CHANGE） ←────────────────────┘            │            │
     │                                                                                                      ↓            ↓
     └─ 想法模糊时先跑议题发现循环                                                                   五层对齐      SHIP / 归档

* 仅前端项目走 2a；后端 / CLI / lib 跳过
** 7-integration 前强制跑（R15 五层对齐）
```

**前端项目路径**：`CHANGE → REQUIREMENT → DESIGN → 2a UI-DESIGN → TASK → DEV → TEST → REVIEW (3 轮) → S-ALIGN → INTEGRATION`
**后端项目路径**：`CHANGE → REQUIREMENT → DESIGN → TASK → DEV → TEST → REVIEW → S-ALIGN → INTEGRATION`
**产品先行路径**（可选）：`D-discovery → P-product（产 PRODUCT-DESIGN.html）→ 1-requirement（AC 引用其 §）→ …`
**MVP 路径**：`REQUIREMENT → DESIGN-lite → TASK → DEV`（阶段可压缩，但关键工件不能缺；`DESIGN-lite` 至少写清技术栈、触碰边界和不做什么）

---

## 横向命令（主流程外按需调用）

不在主流程里，各自独立可用（详见 `README.md` 横向命令表）：

| 命令 | 一句话 | 触发时机 |
|---|---|---|
| `ROUTER` | 语义路由：听懂意图 → 检测工件状态 → 移交对应 prompt | 新会话冷启动 / 说不清要进哪阶段 |
| `D-discovery` | 议题发现循环：事实文档驱动拆子议题 → 逐个讨论 → roadmap（零代码改动 · R16） | "项目调研 / 找改进 / 探索方案" |
| `P-product` | 产品 PR + 界面原型合一：单文件 `PRODUCT-DESIGN.html`（九段制） | "写产品文档 / 出原型" |
| `S-align` | 五层对齐检测：代码/原型/活文档/决策留痕/执行留痕 → 漂移合并（R15） | 7-integration 前（强制）/ M-health 建议 / 手动 |
| `L-restyle` | 一键换调性（UI 不动功能） | 品牌换新 / 暗色版 |
| `M-health` | 代码库周期性巡检（6+6 维 + 冗余巡检） | 月度 / 季度 / 接手陌生项目 |
| `I-intel-scan` | 老项目入场扫描 → CONTEXT.md | brownfield 首次使用 |
| `A-architect` | 项目级架构梳理 → ARCHITECTURE.md | 里程碑后 |
| `A-evolve` | 架构增量同步（DESIGN § 9 → CONTEXT/ARCHITECTURE） | 多 change 后批量同步 |

---

## 五层对齐（R15 · S-align 背后的模型）

一个健康项目的五个真相来源，必须保持一致：

| 层 | 是什么 | 载体 |
|---|---|---|
| L1 代码 | 实际运行的东西 | git 仓库源码 |
| L2 原型 | 界面长什么样 | PRODUCT-DESIGN.html 原型页 / UI-DESIGN.md |
| L3 活文档 | 系统是怎么设计的 | REQUIREMENT / DESIGN / CONTEXT / ARCHITECTURE |
| L4 决策留痕 | 为什么这么做 | DESIGN 内 ADR / CHANGE 范围决策 |
| L5 执行留痕 | 实际做了什么 | SUMMARY / PROGRESS（R4.1 格式） |

漂移判定规则 D1~D6 与合并动作三分详见 `prompts/S-align.md`；每次对齐产出 `.specs/ALIGNMENT.md`（含**覆盖的本地提交列表**与**已对齐记录**两张强制表），作为下一轮的 git 基线。合并动作需用户确认，S-align 本身永不改 L1 代码。

---

## 阶段定义

| 阶段 | 输入 | AI 职责 | 输出文件 | 需人工确认 | 迭代可跳 |
|---|---|---|---|---|---|
| CHANGE | 一句话想法 / bug | 反问澄清 + 影响面判定 | `CHANGE.md` | 是 | 否 |
| REQUIREMENT | CHANGE | 写 AC + 范围切分 + 术语提取 | `REQUIREMENT.md` + `CONTEXT.md` | 是 | 条件性 |
| DESIGN | CHANGE + REQUIREMENT + CONTEXT | **技术栈预选**（5~6 卡让用户选）+ 技术决策 + ADR + 数据流 | `DESIGN.md`（含 `## 0. 技术栈选定`）| 是（栈 + 关键决策）| 条件性 |
| **2a UI-DESIGN** | REQUIREMENT + DESIGN | 美学方向 + design tokens + 反 AI-slop 自检 | `UI-DESIGN.md` | 是（关键决策）| 否（前端必跑）/ 是（非前端跳过）|
| TASK | DESIGN（+ UI-DESIGN）| 拆原子任务 + 标 `[P]` 并行 + 依赖图 | `TASK.md` | 否 | 否 |
| DEV | TASK 中一项 | fresh subagent + TDD + 原子提交 | 代码 + `SUMMARY.md` | 否 | 否 |
| TEST | 代码 + AC + 非功能需求 | **5 轮金字塔**：功能 / 性能 / 安全 / 兼容 / 可观测（按项目类型裁剪）| `TEST.md` | 否 | 否 |
| REVIEW | diff + SPEC | 双轮审查（spec 合规 + 代码质量）| `REVIEW.md` | 是（仅严重项）| 否 |
| S-ALIGN | REVIEW 通过后 · HEAD | 五层漂移检测 + 合并动作提案（R15） | `.specs/ALIGNMENT.md` | 是（合并动作）| 否 |
| INTEGRATION | 全部已通过 | UAT + 集成 smoke + 失败诊断 | `UAT.md` / fix-plan | 是 | 否 |
| ARCHIVE | 已合并 | 折叠 change 进主 spec | `archive/<date>-<name>/` | 否 | — |

---

## 文件体系

所有产物存到：`./.specs/<change-id>/`

| 文件 | 用途 | 谁来写 |
|---|---|---|
| `CHANGE.md` | 一次变更的提案（why / what / 影响面 / 范围排除）| 协作（人起草 + AI 反问补全）|
| `REQUIREMENT.md` | 需求 + 验收准则（用户故事 / AC / v1·v2·out / 非功能性）| AI 主笔，人确认 |
| `CONTEXT.md` | 域语言 + 默认决策（术语表 / 已锁决策 / 偏好）| 协作 |
| `DESIGN.md` | 技术设计（架构图 / 数据流 / ADR / 风险）| AI 主笔，人审 |
| `UI-DESIGN.md` | UI 美学方向 + design tokens（OKLCH 颜色 / 字体 / 间距 / 动效）+ 反 AI-slop 自检（仅前端项目）| AI 主笔，人审 |
| `TASK.md` | 任务清单（任务 / 依赖 / `[P]` / verify / done）| AI |
| `TEST.md` | 5 轮测试金字塔报告（功能矩阵 + UAT + 性能 / 安全 / 兼容 / 可观测口供）| AI |
| `REVIEW.md` | 审查发现（严重度 / 修复决策 / 跨模型分歧）| AI |
| `SUMMARY.md` | 阶段产物快照（用于截断历史）| AI |
| `<task-id>-PROGRESS.md` | **临时**文件——任务执行中途清窗时写入，含「已排除方案」反重复段。任务完成后删除。详见 RULES R1.5/R1.6/R1.7 | AI |
| `LESSONS.md`（`.specs/` 根）| **项目级常驻**——跨 change 失败知识库。每个 DEV 任务开工前必扫；INTEGRATION 阶段提名新条目。详见 RULES R1.8 | AI 提名 + 人工筛 |
| `STATE.md`（仓库根）| 跨会话状态（当前位置 / 中断任务 / 阻塞 / 决策日志）| AI 维护，人可改 |
| `PRODUCT-DESIGN.html`（change 目录）| 产品 PR + 界面原型合一（九段制单文件 HTML · P-product 产出）| AI 主笔，人审 |
| `ALIGNMENT.md`（`.specs/` 根）| 五层对齐报告（覆盖提交 / 漂移 / 合并动作 / 下一轮基线 · S-align 产出）| AI 主笔，合并动作人确认 |
| `discovery/`（change 目录）| 议题发现循环产物（子议题树 / 逐题报告 / roadmap · D-discovery 产出）| AI 主笔，审核门人通过后落盘 |

---

## Agent 角色（同一模型也要扮演不同角色）

| 角色 | 输入 | 输出 | 时机 |
|---|---|---|---|
| **Architect** | CHANGE / REQUIREMENT | DESIGN.md + ADR | DESIGN 阶段 |
| **UI Director** | REQUIREMENT + DESIGN | UI-DESIGN.md（含 design tokens）| 2a UI-DESIGN 阶段（前端项目）|
| **Planner** | DESIGN（+ UI-DESIGN）| TASK.md | TASK 阶段 |
| **Dev**（多实例并行）| TASK 中一项 | 代码 + 任务级 SUMMARY | DEV 阶段，每任务一个 fresh context |
| **Reviewer** | diff + SPEC | REVIEW.md | REVIEW 阶段（建议双轮：同模型 spec 审 + 异模型代码审）|
| **Verifier** | 构建产物 + AC | UAT.md / fix-plan | INTEGRATION 阶段 |

**红线**：Architect 不写代码，UI Director 不写完整组件，Dev 不改 SPEC / UI-DESIGN，Reviewer 不修代码（只产报告 + 修复 task）。

---

## 7 个核心机制（已内化进各阶段 prompt）

1. **任务拆解** — 拆到 fresh context 跑得完且自带 verify 的最小单元，按文件冲突切而不是按层切
2. **Prompt 组织** — 模板 + 文件引用 + 验证锚点；不靠对话堆叠
3. **上下文控制** — 阶段切换 = 清窗 + 重新载入指定 .md
4. **代码生成策略** — 默认分步（每任务 fresh context），仅小特性允许一次性
5. **测试生成** — 测试从 AC 派生而非从实现派生；bug 修复必伴随回归测试
6. **审查 / Refine 循环** — 至少两轮，至少一轮跨模型 spot-check
7. **自迭代** — 失败自动产 fix-plan 回炉，最多 3 轮，超限人工介入

---

## 自迭代上限

任何阶段（plan / dev / verify）的自动重试次数 **≤ 3 轮**。
超限必须停下来让人决策，禁止死循环。
