# GO — code-kit 统一入口（每个 IDE 都用这一个）

> **用户使用方式**：`@code-kit/GO.md` + 一句话意图
> AI 看到 `@GO.md` 就按本文件路由，自动决定阶段、自动生成 ID、自动按需加载工件，**不要等用户提供 ID 或路径**。

---

## 红线·Token 预算（走任何阶段前必读）

code-kit 的文件分两类，**加载策略不同**：

| 类型 | 路径 | 长度 | 加载方式 |
|---|---|---|---|
| **SPEC**（项目产物）| `.specs/<id>/*.md` | 通常 < 200 行 | 整读 OK |
| **REFERENCE**（查阅型）| `code-kit/reference/*.md` | 75~470 行 | **禁止默认整读**，只 grep / read offset 需要的那一节 |
| **TEMPLATE / PROMPT** | `code-kit/templates|prompts/*.md` | ≤ 300 行（R1.10） | 整读 OK |

**违规示例**（AI 常犯）：

- ❌ 进入 2-design 阶段后直接 `read_file code-kit/reference/tech-stacks.md` 整读 467 行
- ✅ 正确：`grep_search` 查 `适用矩阵` 或 `read_file offset=??? limit=80` 只读所需那节

- ❌ 进入 2a-ui-design 后同时读 `ui-aesthetics.md` 与 `ui-anti-patterns.md` 两全文
- ✅ 正确：`ui-anti-patterns.md` 75 行可整读；`ui-aesthetics.md` 只读「调性」一节

**Token 预算**：进入任何阶段的首轮消息，加载的 reference 总行数 ≤ **150 行**。超过 → 拆到后面按需拉。

---

> 新 CHANGE 路由后 → 整读 `@code-kit/go-budget.md`（64 行）输出估算与挡位

---

## 第一步 · 读取项目状态（必须，跳过即违反）

1. 尝试读 `STATE.md`（仓库根）。不存在 → 视为新项目，跳过
2. 关注字段：`活跃 Change` / `当前阶段` / `当前 Task` / `中断任务`
3. 如果存在 `中断任务` 非空 → **优先级最高**，直接走"恢复中断任务"分支（见下表）

### 零外部依赖声明

code-kit 只依赖 markdown 与 git 本身：不检测、不调用任何包管理器、语言运行时、外部设计工具或运行时适配器。所有决策基线（美学 / 反模式 / 测试金字塔 / 界面样例）均内置在 `reference/` 与 `ui-samples/`，断网可用。

## 第二步前 · Artifact Preflight Gate（强制）

路由到任何阶段前，先检查上游工件是否存在且足够用。**阶段可以压缩到同一轮对话里做，但关键工件不能缺席。**

| 目标阶段 | 必须已有的上游工件 | 缺失时动作 |
|---|---|---|
| `0-change` | 无 | 直接进入 |
| `1-requirement` | `.specs/<id>/CHANGE.md` | 回 `0-change` 生成 CHANGE |
| `2-design` | `.specs/<id>/CHANGE.md` + `.specs/<id>/REQUIREMENT.md` | 缺哪个回哪个阶段补齐 |
| `2a-ui-design` | `CHANGE.md` + `REQUIREMENT.md` + `DESIGN.md` | 回缺失阶段补齐；UI 项目不能用极简模式跳过 |
| `3-task` | `REQUIREMENT.md` + `DESIGN.md`；前端/UI 项目还必须有 `UI-DESIGN.md` | 回缺失阶段补齐 |
| `4-dev` | 正式 `.specs/<id>/TASK.md` 中的当前 task，或用户显式提供的临时最小 TASK | 没有就反问：回 `3-task` 生成正式 TASK，还是由用户提供临时最小 TASK |
| `5-test` | `REQUIREMENT.md` + `DESIGN.md` + `TASK.md` + 各 `*-SUMMARY.md` | 回缺失阶段补齐 |
| `6-review` | `REQUIREMENT.md` + `TASK.md` + `TEST.md` + 本次 diff；有 `DESIGN.md` / `UI-DESIGN.md` 时必须一起读 | 回缺失阶段补齐 |
| `7-integration` | `.specs/<id>/` 下本 change 的全部应有产物 | 回缺失阶段补齐 |

临时最小 TASK 不是正式 `TASK.md` 的隐式替代品。它只允许用于单点调用 `4-dev`，且必须由用户显式给出，包含 `id / name / read_files / write_files / action / verify / done`。AI 不允许为了绕过 `3-task` 自己编造临时 TASK。

Preflight 失败时，路由声明必须写明：

```text
规则 R2.7 触发：目标阶段缺少 <工件>。本次先回到 <阶段> 补齐，不能直接继续。
```

## 第二步 · 解析用户意图，路由到阶段

匹配表格前先判断是否是**新事物描述**：如果当前没有活跃 change，且用户是在说"做 / 想 / 加 / 实现 / 设计 + X"，即使句子里含有"设计 / UI / 测试 / review"等词，也优先路由到 `0-change`。只有已经存在活跃 change 且 Artifact Preflight Gate 通过时，才允许直达中间阶段。

按以下表格匹配用户输入（在上面优先级之后，**取最先命中那一条**）：

| 用户输入特征 | 路由到 | 备注 |
|---|---|---|
| `自动化` / `多agent` / `挂机` / `自动提交` / `自动拉齐` / `人工介入` / `哪里需要人审` / `什么时候停` | `AUTOMATION.md` | 多 agent 协作架构：七类 agent / 任务状态机 / 日志判定 / 提交标准 / 澄清式人工介入 + 沉淀前置（R17/R18） / **分阶段扇出编排手册 §9** |
| `bmad` / `BMAD` / `和 BMAD 配合` / `用哪个技能` | `BMAD.md` | BMAD 技能族 ↔ code-kit 集成映射；判断某 bmad-* 技能该不该用、产物落哪 |
| `暂缓` / `下轮再说` / `以后做` / `backlog` / `parking lot` | `AUTOMATION.md` §8 | 暂缓项 = 议题（R18.4）：统一登记为议题，走 D-discovery 循环消化，禁用同义概念 |
| `调研` / `议题讨论` / `看看有啥改进` / `怎么优化` / `竞品分析后给建议` / `探索一下方案` / `评估这个改进想法` | `prompts/D-discovery.md` | 议题发现**前置阶段**（kind: discovery）：事实文档拆子议题 → 逐个五步讨论（GD 门 + ⑤ 人审）→ ROADMAP；议题即 change（R16.5），立项时同目录补 CHANGE.md 续走主干；全程零代码改动（R16） |
| `写产品文档` / `产品 PRD 和原型` / `产品设计文档` / `出一份带原型的需求文档` | `prompts/P-product.md` | 产品 PR + 界面原型合一：单文件 `PRODUCT-DESIGN.html`（九段制）；是 1-requirement 的可选上游 |
| `对齐检查` / `拉齐` / `检查漂移` / `文档和代码是不是偏了` / `留痕补齐` | `prompts/S-align.md` | 五层对齐检测（R15）；7-integration 前强制跑一次 |
| `继续` / `接着上次` / `恢复` / `resume` | `prompts/4-dev.md` 的「入场恢复」段 | 加载 `STATE.md` 中断任务对应的 PROGRESS |
| `执行 T<NN>` / `跑 T<NN>` / `do T<NN>` | `prompts/4-dev.md` | task-id 从用户输入提取 |
| `审查` / `review` / `检查代码` / `code review` | `prompts/6-review.md` | |
| `测试` / `写测试` / `UAT` / `test` | `prompts/5-test.md` | |
| `上线` / `集成` / `验收` / `ship` / `归档` | `prompts/7-integration.md` | |
| `拆任务` / `plan tasks` / `分解` | `prompts/3-task.md` | |
| `设计` + `<已有需求>` / `架构` / `design` | `prompts/2-design.md` | 仅当 `CHANGE.md` + `REQUIREMENT.md` 已存在；否则回缺失阶段 |
| `选技术` / `选栈` / `选框架` / `tech stack` / `用什么开发` / `迁移评估` | `prompts/2-design.md` 步骤 0 | 只需技术栈选型时入口 |
| `UI` / `视觉` / `美学` / `theme` / `design system` / `design tokens` | `prompts/2a-ui-design.md` | 前端项目，用户可见 UI；必须已有 `CHANGE.md` + `REQUIREMENT.md` + `DESIGN.md` |
| `换调性` / `改风格` / `换风格` / `redesign` / `restyle` / `重做视觉` / `换皮` | `prompts/L-restyle.md` | 已有项目换视觉，保留功能 |
| `健康检查` / `health` / `体检` / `技术债扫描` / `巡检` / `扫冗余` / `找死代码` / `找重复` / `清未用导出` / `清未用依赖` / `dedupe` / `dead code` | `prompts/M-health.md` | 代码库周期性巡检 + 冗余扫描（步骤 2.5），不属任何 change |
| `扫描代码` / `scan` / `intel` / `入场扫描` / `给项目体检` / `老项目首次访问` | `prompts/I-intel-scan.md` | 生成 / 更新 `.specs/CONTEXT.md`，brownfield 项目首次使用必跑 |
| `同步架构` / `沉淀架构` / `evolve` / `架构演进` / `同步 CONTEXT` / `整理沉淀` | `prompts/A-evolve.md` | 扫近期归档 change 的 DESIGN § 9，批量 review + patch CONTEXT.md / ARCHITECTURE.md（不属任何 change）|
| `建立架构` / `架构梳理` / `重构架构` / `architect` / `重审 ADR` / `画架构图` | `prompts/A-architect.md` | 首次 / 重构时建立 `ARCHITECTURE.md`。项目级 ADR / 模块图 / 跨模块契约（不属任何 change）|
| `需求` / `spec` / `requirement` | `prompts/1-requirement.md` | |
| 任何**新事物描述**（"做 / 想 / 加 / 实现 / 设计 + X"，且当前无活跃 change） | `prompts/0-change.md` | **自动生成 change-id**，不要问用户要 |
| 模糊不清 | 反问用户：「你想做的是新需求 / 继续上次 / 别的吗？」 | 一句话定位 |

> **架构级变更的二次拦截**：路由表只看关键词，但 0-change / 2-design 进去后会**用判断**做二次检测——如果用户描述涉及「拆模块 / 换数据库 / 换鉴权方案 / 改公共契约 / 容量边界 / 跨服务编排」这类项目级变更，会停下来反问"是否先跑 A-architect"。判定细则见 `@code-kit/prompts/0-change.md` 步骤 0.4 / `@code-kit/prompts/2-design.md` 步骤 0₋。
>
> **AI 的额外职责**：路由到 0-change / 2-design 时，路由声明的「第一动作」一栏应明确写"按 0.4 / 0₋ 先做架构级预检"，避免遗漏。

## 第三步 · 老项目入场检测（brownfield 必跑）
STATE 无 CONTEXT/技术栈行且 .specs 不存在时 → 整读 `@code-kit/go-entry-check.md`（91 行）执行 3.1–3.4；命中任一豁免（R1.10 冷启动例外）直接跳过

## 第四步 · 自动准备（不打扰用户）

进入对应阶段前，AI 必须自行完成：

- **新 CHANGE**：按 `prompts/0-change.md` 的步骤 0 自动生成 `change-id`（kebab-case，2~4 词），并在第一条回复里显式声明
- **目录不存在**：自行 `mkdir -p .specs/<id>/`，不要让用户先建
- **规则加载**：若 IDE 未注入全局规则，读 `@code-kit/RULES.md`（精简版 `@code-kit/SYSTEM.md` 也行）
- **门禁计票 / 角色装配 / 法定停点核对** → 整读 `@code-kit/go-gates.md`（43 行）；各 prompt 内联的 🛡️ 门禁段仍是执行格式事实源
- **路由声明**：用户话语是自然语言而非阶段号时，先走 `@code-kit/prompts/ROUTER.md` 语义路由（含调研 `D-discovery` / 产品文档 `P-product` / 五层对齐 `S-align` 三个新入口）

### 加载工件（严格区分 必读 / 按需）

**语义约定**：
- `⚡︎ 全读` ：进阶段首轮必须 read_file 整个文件（只出现在 SPEC / TEMPLATE 上）
- `⚡︎ 查表` ：只 grep 指定节 或 read offset/limit，**禁止默认整读**（reference/* 都是这个）
- `⚡︎ 按需` ：首轮不读，里面某个决定点需要时才 grep / 读

| 阶段 | 全读（SPEC） | 查表（REFERENCE，只读指定节） | 按需 |
|---|---|---|---|
| 0 / 1 | —（新建）| `code-kit/reference/ui-aesthetics.md` 只查「给 AI 在 0-change 阶段展示用的标准模板」一节（仅前端项目）| — |
| 2 | `<id>/CHANGE.md` + `<id>/REQUIREMENT.md` + `.specs/CONTEXT.md` + `.specs/ARCHITECTURE.md`（如存在 · brownfield 强烈推荐 · 重点读 § 2/§ 3/§ 4）| `code-kit/reference/tech-stacks.md` 只查「适用矩阵」+ 过滤出的 5~6 张卡片 | ADR 阶段某项要深谈时再读 |
| 2a | `<id>/CHANGE.md` + `<id>/REQUIREMENT.md` + `<id>/DESIGN.md` `## 0` 段 + `.specs/CONTEXT.md` + `code-kit/reference/ui-anti-patterns.md`（仅 75 行可全读）| `code-kit/reference/ui-aesthetics.md` 查「5 维度」+ 「给 AI 的模板」 | `ui-samples/INDEX.md` 挑基线（内置零依赖）|
| 3 | `<id>/REQUIREMENT.md` + `<id>/DESIGN.md` + `<id>/UI-DESIGN.md`（前端项目）+ `.specs/CONTEXT.md` | — | 任务模板查询 |
| 4 | `<id>/TASK.md`（只读当前 task 块）+ `<id>/DESIGN.md` `## 0` 段 + `<id>/UI-DESIGN.md`（UI 任务）+ `.specs/CONTEXT.md` + `.specs/LESSONS.md` | `code-kit/reference/ui-anti-patterns.md`（UI 任务 · 75 行可全读）| — |
| 5 | `<id>/REQUIREMENT.md` + `<id>/DESIGN.md` `## 0` 段 + `<id>/TASK.md` + 各 `*-SUMMARY.md` | `code-kit/reference/test-pyramid.md` 只查「适用矩阵」+ 需要的那几轮详情 | — |
| 6 | `<id>/REQUIREMENT.md` + `<id>/DESIGN.md` + `<id>/TASK.md` + `<id>/TEST.md` + `git diff` | `code-kit/reference/ui-anti-patterns.md`（前端项目第三轮 · 75 行可全读）| — |
| 7 | `.specs/<id>/` 全部产物 + `.specs/LESSONS.md` | — | — |
| **M** (health) | `.specs/CONTEXT.md` + `.specs/LESSONS.md` + 最近 1 份 `.specs/health/*.md`（如有，做对比基线）| — | 抽样 5 个最近改动频繁的 src/ 模块 + 5 个测试文件 + 最近 30 天 git log |
| **A** (evolve) | `STATE.md` + `.specs/CONTEXT.md` + `.specs/ARCHITECTURE.md`（如存在）+ 范围内每个 `.specs/archive/<change>/DESIGN.md` 的 § 9 段（仅 § 9，非整份 DESIGN）| — | 仅扫 `last_evolve_at` 之后归档的 change，禁止越界读 § 9 以外的 DESIGN 内容 |
| **A** (architect) | `.specs/CONTEXT.md` + `.specs/ARCHITECTURE.md`（如存在）+ `.specs/CHANGELOG.md` + `code-kit/templates/ARCHITECTURE.md`（模板）| — | `src/` 顶层结构 + `package.json` / 依赖文件 + 抽样几份 `.specs/archive/*/DESIGN.md` |

### 工程底座参考（新增 · 按需查节，不整读）

| 主题 | 文件 | 何时用 |
|---|---|---|
| TDD / 增量实现 / 调试 / API / 文档 / Git / 审查方法 / 废弃迁移 | `reference/engineering.md` | 实现、修 bug、接口设计、写 ADR、提交与发版、删旧代码 |
| 安全 / 性能 / 可观测 / 五轮测试 / UI 基线 | `reference/quality.md` | 5-test、涉及输入/鉴权/数据、性能优化、上线观测 |
| 质询 / 调研 / PRD / UX / 任务拆解 | `reference/planning.md` | 0-change 之前想清楚、P-product、拆任务 |
| ponytail 最小完整改动 / diff 审查 / 全仓审计 / 债务台账 | `reference/ponytail.md` | 4-dev 实现、6-review 审查、M-health 审计 |
| 多 agent 自动化协作 | `AUTOMATION.md`（主干协议，不在 reference 计数内） | 挂机 / 多 agent / 提交与拉齐 |
| 能力来源与覆盖 | `reference/coverage.md` | 追溯"这个能力从哪来" |

规则依据：`@code-kit/RULES-FLOW.md` R17（多 agent 协作）/ R18（澄清与沉淀）/ R19（阶梯律·最少实现）；R1–R8 见 `RULES.md`。

### 查 reference 某一节的实际动作示例

```
# ± 查「适用矩阵」那一节的起始行
grep_search Query="适用矩阵" SearchPath="code-kit/reference/tech-stacks.md"
# 取到 line 380 左右为起始，再：
read_file path="code-kit/reference/tech-stacks.md" offset=380 limit=60
```

不要「为了保险」一上来就整读。不仅费 token，还让你在后面的推理中被无关节况干扰。

## 第五步 · 显式声明执行计划（必须）

进入实际工作前，AI 必须输出一段**路由声明**，**每个加载项必须标明起止行或「全读」**：

```
✅ 路由：<阶段，例如 0-change>
✅ Change-ID：<id>（已自动生成 / 已恢复活跃 change：<existing-id>）
✅ 已加载：
   - <file1>（全读，N 行）
   - <file2>（全读，N 行）
   - <reference-file>（仅查「某节」，line X-Y）
✅ 未加载：<本阶段不需但后面可能用到的 reference，明说何时才拉>
✅ 第一动作：<具体下一步，例如 "按 0-change 流程反问澄清，先问 3 个问题"）
```

示例（2-design 阶段首轮）：

```
✅ 路由：2-design
✅ Change-ID：companion-platform
✅ 已加载：
   - .specs/companion-platform/CHANGE.md（全读，52 行）
   - .specs/companion-platform/REQUIREMENT.md（全读，98 行）
   - .specs/CONTEXT.md（全读，41 行）
   - code-kit/reference/tech-stacks.md（仅查「适用矩阵」，line 380-405，提取出 5 张候选卡）
✅ 未加载：ui-aesthetics.md / test-pyramid.md / ui-anti-patterns.md（后面阶段才需）
✅ 第一动作：按 2-design 步骤 0，列 5 张技术栈卡片 + 推荐 + 排除，等用户选定后才出 ADR。
```

用户看到这段后可以一句话纠偏（"换 id"、"我想要的是别的阶段"、"你加载太多了"），AI 必须接受。

## 第六步 · 执行对应阶段 prompt

加载 `prompts/<n>-*.md` 的内容并按其指令推进。所有规则（`RULES.md` / `SYSTEM.md`）继续生效。

---

## 极少数情况：用户根本没说他想做什么

例：用户只发了 `@code-kit/GO.md`，正文空。

→ AI 必须主动反问，给出 3 个最可能的选项让用户选：
1. 我有个新想法想做
2. 继续上次的工作（如果 STATE 有活跃 change，主动列出）
3. 我要审查/测试某段已有代码

不要瞎猜路由。

---

## 自检（产出路由声明前）

- [ ] 已读 STATE.md（如果存在）
- [ ] 已按表格匹配意图，没有跳过
- [ ] 新 CHANGE 已自动生成 ID 并展示
- [ ] **Token 预算**：本轮加载的 reference/* 总行数 ≤ 150（全读仅 ui-anti-patterns 75 行 · 其他均查节）
- [ ] **未越界**：没有读上表「查表」或「按需」列中的文件为全文
- [ ] 路由声明含「已加载 / 未加载 / 起止行」三要素
- [ ] 没有要求用户提供 ID / 路径 / 阶段名（这些 AI 自己决定）
