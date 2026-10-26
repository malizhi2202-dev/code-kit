# 方法论（code-kit 骨架 · AI 必读）

> **你是 AI 助手。这是 code-kit 的方法论全文，每次会话开始 / 清窗恢复时必读一遍。**
> 读完之后你应该知道：当前处于哪个阶段、要产出什么工件、和其他阶段的输入输出关系、哪些核心机制不允许绕过。
> 后续具体执行按各阶段 `prompts/<n>-*.md` 的指令走，本文件只定义骨架。

---

## 标准流程

```
[DISCOVERY]* → CHANGE → REQUIREMENT → DESIGN → [2a UI-DESIGN]* → TASK → DEV → TEST → REVIEW → [S-ALIGN]** → INTEGRATION → ARCHIVE
     │             │          │           │            │            │      │       │         │            │            │
     │             │          └─ CONTEXT.md 跨阶段共享 ┘     前端项目 ┘     └ TDD ─┘         │            │            │
     │             │                                                                      │            │            │
     │             └─────────────────────────── 迭代回灌（new CHANGE） ←────────────────────┘            │            │
     │                                                                                                      ↓            ↓
     └─ 想法模糊先进议题发现前置阶段（kind: discovery · 议题即 change，过审后同目录续 CHANGE）                                    五层对齐      SHIP / 归档

* 条件阶段：DISCOVERY 仅议题先行时跑（想法模糊/系统性找改进——议题即 change，R16.5）；2a 仅前端项目，后端 / CLI / lib 跳过
** 7-integration 前强制跑（R15 五层对齐）
```

**前端项目路径**：`CHANGE → REQUIREMENT → DESIGN → 2a UI-DESIGN → TASK → DEV → TEST → REVIEW (3 轮) → S-ALIGN → INTEGRATION`
**后端项目路径**：`CHANGE → REQUIREMENT → DESIGN → TASK → DEV → TEST → REVIEW → S-ALIGN → INTEGRATION`
**议题先行路径**（可选前置阶段，R16.5 议题即 change）：`D-discovery（同一 change：TOPICS.md → 五步逐子议题 → ROADMAP.md 过审）→ CHANGE.md（同目录补写）→ …`
**产品先行路径**（可选）：`D-discovery → CHANGE.md → P-product（产 PRODUCT-DESIGN.html）→ 1-requirement（AC 引用其 §）→ …`
**MVP 路径**：`REQUIREMENT → DESIGN-lite → TASK → DEV`（阶段可压缩，但关键工件不能缺；`DESIGN-lite` 至少写清技术栈、触碰边界和不做什么）

---

## 横向命令（按需调用）

`D-discovery` / `P-product` 是主干的**可选前置阶段**（已进下方阶段定义表与 GO.md 门禁总览，工件落 change 目录）；下表其余命令在主流程外，各自独立可用（详见 `README.md` 横向命令表）：

| 命令 | 一句话 | 触发时机 |
|---|---|---|
| `ROUTER` | 语义路由：听懂意图 → 检测工件状态 → 移交对应 prompt | 新会话冷启动 / 说不清要进哪阶段 |
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

## 自动化协作与人工介入（R17/R18）

**能自动的全部自动；人工只在需要澄清的地方介入，阶段推进的唯一硬前置是上一阶段先沉淀输出物**（详见 `AUTOMATION.md`）：

- **多 agent 分工**：前 3 阶段（CHANGE / REQUIREMENT / DESIGN[+2a]）相互依赖，由主 agent 顺序完成；3-task 拆解后才切给并行子 agent（波次）。七类 agent：主拆解 / 子开发 / 测试 / review / 日志扫描 / 自动提交 / 自动拉齐（S-align 常驻化）。
- **任务流水线**：`done` ≠ 完成——子 agent 自检 done → 测试（每完成一个就测，不等波次）→ review → 待提交；日志扫描 agent 判定错误，done 且无依赖且产物完整可读 → 进修改阶段。
- **沉淀前置（R18.3）**：进入下一阶段前，上一阶段输出物必须已落盘（工件 + STATE.md 状态行）——对话记忆不算数；与 R2 Artifact Preflight 构成双向闭环。
- **人工介入是触发式，不是固定关卡**：上下衔接清楚、方案清楚的阶段**直通推进（落盘即走）**；只在四类澄清触发时停——① 歧义（调研 4-5 套方案拍板，自动 60%/人工 40%）② 偏好（UI 风格与逐页排版，50%/50%）③ 纠正（人看原型/界面/测试不满意 → 重生成，循环到满意；不主动等人）④ 把关（git 提交、破坏性变更落盘前）。澄清一次性问清，其余禁止"要不要继续？"式提问。
- **暂缓项 = 议题**：所有"先不做/下轮再说"统一登记为议题（R18.4：落议题 change 的 `TOPICS.md` 状态表或 STATE.md 留档行），由 D-discovery 前置阶段消化。
- **BMAD 集成**：BMAD 技能族作为各阶段深挖工具接入，映射见 `BMAD.md`。

---

## 阶段定义

| 阶段 | 输入 | AI 职责 | 输出文件 | 需人工确认 | 迭代可跳 |
|---|---|---|---|---|---|
| **D-DISCOVERY**（可选前置 · R16） | 事实文档（项目文档 / 代码 / 配置 / 数据 / 用户原话） | 第 0 步拆子议题（每条带依据）→ 逐子议题五步：①头脑风暴 ②竞品侦察 ③结论建议 ④GD 门（3 人减配）⑤人审门；随步落盘、状态标晋级 | `TOPICS.md` + `D-<子议题slug>.md` + `ROADMAP.md` | 是（⑤ 逐子议题，法定停点） | 是（想法已清晰 → 直接 CHANGE）|
| **P-PRODUCT**（可选前置） | CHANGE.md 或 ROADMAP.md | 产品 PR + 界面原型合一（九段制 · 单文件零依赖 HTML） | `PRODUCT-DESIGN.html` | 是（文档与原型人审） | 是 |
| CHANGE | 一句话想法 / bug / **同 change 的 ROADMAP.md** | 反问澄清 + 影响面判定 | `CHANGE.md` | 是 | 否 |
| REQUIREMENT | CHANGE | 写 AC + 范围切分 + 术语提取 | `REQUIREMENT.md` + `CONTEXT.md` | 是 | 条件性 |
| DESIGN | CHANGE + REQUIREMENT + CONTEXT | **技术栈预选**（5~6 卡让用户选）+ 技术决策 + ADR + 数据流 | `DESIGN.md`（含 `## 0. 技术栈选定`）| 是（栈 + 关键决策）| 条件性 |
| **2a UI-DESIGN** | REQUIREMENT + DESIGN | 美学方向 + design tokens + 反 AI-slop 自检 | `UI-DESIGN.md` | 是（关键决策）| 否（前端必跑）/ 是（非前端跳过）|
| TASK | DESIGN（+ UI-DESIGN）| 拆原子任务 + 标 `[P]` 并行 + 依赖图 | `TASK.md` | 否 | 否 |
| DEV | TASK 中一项 | fresh subagent + 阶梯律（R19）+ TDD + 原子提交 | 代码 + `SUMMARY.md` | 否 | 否 |
| TEST | 代码 + AC + 非功能需求 | **5 轮金字塔**：功能 / 性能 / 安全 / 兼容 / 可观测（按项目类型裁剪）| `TEST.md` | 否 | 否 |
| REVIEW | diff + SPEC | 双轮审查（spec 合规 + 代码质量）| `REVIEW.md` | 是（仅严重项）| 否 |
| S-ALIGN | REVIEW 通过后 · HEAD | 五层漂移检测 + 合并动作提案（R15） | `.specs/ALIGNMENT.md` | 是（合并动作）| 否 |
| INTEGRATION | 全部已通过 | UAT + 集成 smoke + 失败诊断 | `UAT.md` / fix-plan | 是 | 否 |
| ARCHIVE | 已合并 | 折叠 change 进主 spec | `archive/<date>-<name>/` | 否 | — |

---

## 文件体系

所有产物存到：`./.specs/<change-id>/`

> **体量红线**：单工件超 250 行 / 25 KB → 拆同名文件包（目录 + 语义名成员 + 索引承载状态），发现类细则见 R16.8；面向人整体阅读的单文件（PRODUCT-DESIGN.html）豁免。

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
| `TOPICS.md`（change 目录）| 议题申报 + 子议题状态表（slug / 依据 / 轮次 / 审核标）+ 轮次记录 · D-discovery 第 0 步产出 | AI 主笔，随步更新 |
| `D-<子议题slug>.md` 或超量时的文件包 `D-<子议题slug>/`（TOPIC 索引 + BRAINSTORM/RECON/CONCLUSION/PANEL/REVIEW · R16.8）| 子议题五步报告（头部状态标 `审核: 待审\|通过\|否决（第N轮）`，"通过"须引用户原话+时间）| AI 随步落盘，状态由 ⑤ 门人审改写 |
| `ROADMAP.md`（change 目录）| **只汇总已过审结论** + 《建议改动清单》+ 落地顺序 | AI，每次过审更新 |

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

## 8 个核心机制（已内化进各阶段 prompt）

1. **任务拆解** — 拆到 fresh context 跑得完且自带 verify 的最小单元，按文件冲突切而不是按层切
2. **Prompt 组织** — 模板 + 文件引用 + 验证锚点；不靠对话堆叠
3. **上下文控制** — 阶段切换 = 清窗 + 重新载入指定 .md
4. **代码生成策略** — 默认分步（每任务 fresh context），仅小特性允许一次性
5. **测试生成** — 测试从 AC 派生而非从实现派生；bug 修复必伴随回归测试
6. **审查 / Refine 循环** — 至少两轮，至少一轮跨模型 spot-check
7. **自迭代** — 失败自动产 fix-plan 回炉，最多 3 轮，超限人工介入
8. **最少实现（R19 阶梯律）** — 最好的代码是根本没写的那行：先问需不需要存在（投机=不写），再本仓已有 → 标准库 → 平台原生 → 已装依赖 → 一行 → 最小实现；新依赖须证明前几级不覆盖；有意切角留 `ponytail:` 标记由 M-health 收割。阶梯只缩短方案，绝不缩短理解

---

## 自迭代上限

任何阶段（plan / dev / verify）的自动重试次数 **≤ 3 轮**。
超限必须停下来让人决策，禁止死循环。
