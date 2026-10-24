---
name: code-kit
description: 一站式研发交付工作流：意图路由 → 0-change…7-integration 八阶段闭环 → 8 道专家门禁 → 逐任务自动化投票；内置 ponytail 最小完整改动律、16 个工程技能（TDD/调试/安全/性能/API/文档/Git/迁移等）与 DSH 工具接线。当用户要求从想法到交付完整开发、写需求/设计/拆任务/实现/测试/审查，或要求少写代码、别过度设计时使用。
whenToUse: 用户要求从想法到交付完整开发、写需求/设计/拆任务/实现/测试/审查、老项目入场/体检/对齐，或明确要求"少写代码/别过度设计/最省实现"
short_description: Code-kit software delivery workflow.
short_description_zh: code-kit 研发交付工作流。
version: 2
updated: 2026-10-24T00:00:00Z
---

# code-kit

一套把「想法 → 可交付代码」压成固定轨道的研发流程，外加一组工程硬规矩。
本文件是 **DSH 技能入口**：负责路由、总律与加载；逐阶段细节在 `GO.md` / `prompts/`，工程底座在 `reference/`。

**仓库根目录 = 技能根目录**。DSH 安装后相对路径直接可用；把本目录复制进项目当 `code-kit/` 用时，路径加 `code-kit/` 前缀。

---

## 0. 三条总律（压过一切细节）

1. **最小完整改动（ponytail）**：最好的代码是没写的代码。先问「要不要存在」，再问「仓库里有没有」，再问「标准库/平台/已装依赖能不能做」，最后才写最少的新代码。**省的是方案，不是改动本身**——调用方、测试、夹具、配置该改全改。
2. **绝不省的**：信任边界的校验、防止数据丢失的错误处理、安全、无障碍、真实硬件需要的校准、用户明确要求的东西、以及非平凡逻辑的一条小测试。缺一条都算没做完。
3. **证据而非声称**：没跑过 `verify` 不许说"完成"；没贴出命令输出不许说"通过"；没 grep 过不许说"项目里没有"。

---

## 1. 先路由（不要等用户报阶段号）

### 1.1 读状态（强制第一步）

1. 读仓库根 `STATE.md`（有则读）：`活跃 Change` / `当前阶段` / `当前 Task` / `中断任务`。
2. `中断任务` 非空 → 恢复：先读对应 `PROGRESS.md` 的「已排除方案」，按 `RULES.md` R1.5/R1.6 续跑，**禁止重新规划整个任务**。
3. 无 `.specs/` 或用户描述的是全新事物 → 从 `0-change` 起步。

### 1.2 意图 → 入口（完整语义路由表见 `GO.md` 第二步）

| 用户说的话 | 走哪 |
|---|---|
| 新功能 / 新想法 / "加个 X" / "实现 X" | `prompts/0-change.md`（自动生成 change-id） |
| 写需求 / AC / 验收标准 | `prompts/1-requirement.md` |
| 技术方案 / 选型 / 架构 / ADR | `prompts/2-design.md` |
| 界面 / 视觉 / design tokens / 调性 | `prompts/2a-ui-design.md`（前端项目） |
| 拆任务 / 排期 / 任务清单 | `prompts/3-task.md` |
| 写代码 / 修 bug / 实现某个 task | `prompts/4-dev.md`（必须先有 TASK） |
| 测试 / UAT / 测试矩阵 | `prompts/5-test.md` |
| 审查 / review / 查风险 | `prompts/6-review.md`（三轮协议） |
| 审查单个 diff / ponytail review / 找可删复杂度 | `reference/ponytail.md` §5（四段格式，不走三轮协议） |
| 全仓审计 / "我继承了这个仓库" | `reference/ponytail.md` §6；`prompts/M-health.md` |
| 捷径台账 / 技术债台账 / shortcut 注释 | `reference/ponytail.md` §7 |
| 少写点 / 再狠点 / 停用 ponytail | `reference/ponytail.md` §0（等级与开关） |
| 继续 / 接着上次 / resume / 执行 T<NN> | 恢复：`STATE.md` + `TASK.md` 当前 task + `PROGRESS.md`（R1.5/R1.6） |
| 调试 / 为什么崩了 / 性能回归 | `reference/engineering.md` §2（先造能红的回路） |
| 文档 / ADR / README / changelog | `reference/engineering.md` §4 |
| 慢 / 安全 / 依赖审计 / 弃用迁移 | `reference/quality.md` §1–2；`reference/engineering.md` §7 |
| 集成 / 收尾 / 归档 / 上线 | `prompts/7-integration.md`（进阶段后第一步跑 S-align） |
| 调研 / 议题讨论 / 看看怎么改进 | `prompts/D-discovery.md` |
| 老项目第一次接入 / 摸底 | `prompts/I-intel-scan.md` |
| 体检 / 查技术债 / 找死代码 | `prompts/M-health.md` |
| 换风格 / 重做视觉 | `prompts/L-restyle.md` |
| 对齐 / 文档和代码偏了 | `prompts/S-align.md` |
| 架构梳理 / 沉淀架构 | `prompts/A-architect.md` / `prompts/A-evolve.md` |
| 产品 PRD + 原型 / UX 规格 | `prompts/P-product.md`；`reference/planning.md` §3 |
| BMAD / PRD / epic / story / sprint / 复盘 | `prompts/B-bmad-extensions.md` |
| 重大变更影响 / correct course | `prompts/C-course-correction.md` |
| 想不清楚，先问明白 | `reference/planning.md` §1（设计树访谈） |

**规模速判**（档位细则见 `GO.md` 的 Token 预算段）：
- < 50 行 / typo / bugfix → 不走闭环，直接改 + 回归测试 + 自审。
- 50–150 行 / 单会话 → **压缩路径** `0→3→4→6`：允许在 `CHANGE.md` 内写「最小需求/AC」与「最小设计/触碰模块」两节并显式声明豁免，`6-review` 可只交功能轮 + 回归登记。
- 150 行以上 / 多会话 / 对外交付 → 完整或极简流程。

### 1.3 路由前预检（Artifact Preflight）

| 目标阶段 | 必须有 | 缺了怎么办 |
|---|---|---|
| 0-change | 无 | 直接进 |
| 1-requirement | `CHANGE.md` | 回 0-change |
| 2-design | `CHANGE.md` + `REQUIREMENT.md` | 回缺失阶段 |
| 2a-ui-design | 上述 + `DESIGN.md` | 回缺失阶段；UI 项目不许跳 |
| 3-task | `REQUIREMENT.md` + `DESIGN.md`（前端另需 `UI-DESIGN.md`） | 回缺失阶段；**压缩路径**可在 CHANGE 内写最小需求/AC 与最小设计/触碰模块并声明豁免 |
| 4-dev | `TASK.md` 中的当前 task | 回 3-task；只有用户显式给临时最小 TASK 才可单点执行 |
| 5-test | `REQUIREMENT.md` + `DESIGN.md` + `TASK.md` + 各 `SUMMARY.md` | 回缺失阶段 |
| 6-review | 上述 + `TEST.md` + 本次 diff | 回缺失阶段；**压缩路径**可只交功能轮 + 回归登记并声明 |
| 7-integration | 本 change 全部应有产物；`ALIGNMENT.md` 基线覆盖 HEAD | 缺产物回缺失阶段；基线缺/落后 → **进阶段后第 0 步跑 S-align**，不在 Preflight 拦截 |

阶段可以压进同一轮对话，**工件不能缺席**。缺了就说：
`规则 R2.7 触发：目标阶段缺少 <工件>。先回 <阶段> 补齐，不能直接继续。`

### 1.4 路由声明（每次进入阶段先说清）

```
✅ 路由：<阶段>
✅ Change-ID：<id>（新生成 / 恢复 <id>）
✅ 已加载：<文件>（全读 N 行 / 仅查<节> line X-Y）
✅ 未加载：<后面才需要的参考，说明何时拉>
✅ 第一动作：<下一步具体动作>
```

### 1.5 加载预算

- 产物（`.specs/<id>/*.md`）与模板（`templates/*.md`）：可整读。
- `reference/*.md`：**首轮 ≤ 150 行**，先 grep 定位节，再 `read offset/limit`；禁止"为了保险"整读。
- `prompts/<阶段>.md`（< 150 行）可整读；`GO.md` / `RULES.md` 按节读。

---

## 2. 阶段地图（0–7 八阶段 + 2a 子阶段，共 9 个关卡）

| 阶段 | 干什么 | 产出 | 出口门（4 人专家团） |
|---|---|---|---|
| 0-change | 把模糊想法问清楚、划边界 | `CHANGE.md` | G1 需求门 |
| 1-requirement | 用户故事 + 可验证 AC + 域语言 | `REQUIREMENT.md`、`CONTEXT.md` | 需求门 |
| 2-design | 技术栈、决策、ADR、风险、架构对齐 | `DESIGN.md`、`adr/*.md` | G2 方案门 |
| 2a-ui-design | 调性、tokens、组件规约、占位符策略 | `UI-DESIGN.md` | G2a UI 门 |
| 3-task | 原子任务 + 边界 + verify + 波次 + `<auto>` | `TASK.md` | Task 门 |
| 4-dev | 逐任务实现、迁移、边界自查、提交 | 代码 + `<task-id>-SUMMARY.md` | G3 代码门 |
| 5-test | 五轮金字塔（功能/性能/安全/兼容/可观测） | `TEST.md` | 测试门 |
| 6-review | 三轮审查（spec/质量/UI）+ 触发式第四轮 + 修复任务 | `REVIEW.md`、`T-FIX-NN` | G4 审查门 |
| 7-integration | 对齐、全量自动化、UAT、LESSONS、归档 | `UAT.md`、归档、CHANGELOG/STATE 更新（`ALIGNMENT.md` 由 S-align 产出） | — |

门禁构成、投票格式、裁决阈值见 `GO.md` 的「专家团门禁总览」与 `RULES.md` R9/R13；全票/多数通过即自动前进，**禁止再问"要继续吗"**。逐 task 的 `<auto>` 由研发负责人 + 测试 + 安全投票，平票默认 👤（安全优先）。

---

## 3. ponytail 模式（默认开启）

每次实现前按阶梯取**第一个能完整工作**的选项：

1. **需要存在吗？** 没人要的功能、选项、灵活性：砍掉，一行说明。
2. **仓库里已有吗？** helper / 组件 / service / 模式：按周围代码的用法复用。写之前先 grep。
3. **标准库能做吗？** 用。
4. **平台原生能做吗？** `<input type="date">` 优于日期选择器库，CSS 优于 JS。
5. **已安装依赖能做吗？** 用；**绝不为几行代码加新依赖**。
6. **能一行让读者一眼看懂吗？** 一行。
7. 否则：能工作的最少代码。

- 删除优于新增；无聊优于聪明；保持仓库既有分层与约定。
- 修 bug = 修根因：改之前 grep 所有调用方，在共享函数里修一次；只补 ticket 点到的那条路径 = 第二个 bug。
- 有意走捷径且知道上限 → 写 `shortcut: <上限>, <何时升级>` 注释。
- 回复以代码为主；结尾最多三行：跳过了什么 / 何时再加 / 有什么风险。用户要的报告不受此限。

完整规则、diff 审查（ponytail review）、全仓审计（ponytail audit）、债务台账（ponytail debt）、等级（lite/full/ultra）见 `reference/ponytail.md`（原项目斜杠命令名仅作别名）。

---

## 4. 工件与目录（最小集）

```
<repo>/
├── STATE.md                      # 跨会话状态（活跃 change / 阶段 / 任务 / 中断任务）
├── AGENTS.md / CLAUDE.md         # 项目级 AI 约定（可被 STATE.ai_context_doc 指定为上下文源）
└── .specs/
    ├── CONTEXT.md                # 项目上下文、术语表、已锁决策
    ├── ARCHITECTURE.md           # 项目级架构 / ADR / 模块契约（brownfield 推荐）
    ├── LESSONS.md                # 跨 change 失败教训库（常驻，不归档）
    ├── CHANGELOG.md
    ├── adr/                      # NNN-<title>.md
    ├── health/YYYY-MM-DD.md      # M-health 体检报告
    ├── <change-id>/              # CHANGE / REQUIREMENT / DESIGN / UI-DESIGN / TASK
    │                             #   / <task>-SUMMARY / TEST / REVIEW / UAT / ALIGNMENT
    └── archive/<YYYY-MM-DD>-<change-id>/
```

模板见 `templates/`（15 个）：CHANGE / REQUIREMENT / DESIGN / UI-DESIGN / TASK / SUMMARY / PROGRESS / TEST / REVIEW / STATE / LESSONS / ALIGNMENT / ARCHITECTURE / CONTEXT / PRODUCT-DESIGN。
**对话记忆不算数**：决定必须落盘（R18），下一阶段只信工件。

---

## 5. 工程准则（贯穿所有阶段）

- **TDD**：RED（先写会失败的测试，看到它失败）→ GREEN（最少代码通过，看到它通过）→ REFACTOR。bug 先写复现测试。
- **测试从 AC 派生**，不从实现派生；断言行为，不断言内部调用；宁可 DAMP 不要过度 DRY；mock 只用在慢/不确定的边界。
- **复用先于新建**：写任何能力前 grep `utils|helpers|lib|shared|services|hooks`，找到就 import；SUMMARY 里贴 grep 命令与结果。
- **边界**：提交前 `git diff --name-only` 与 TASK 的 `write_files` 比对，越界必须回滚或显式扩范围。
- **schema 变更**：model + 可逆迁移（up/down）同批；检测到 DB 凭据先问是否执行；改字段走 expand/contract，不原地改名。
- **破坏性变更**（删 ≥ 5 行 / 改公共签名 / 改 API / 删文件）：先 grep 引用图 → 反问用户 → 回归测试覆盖，缺一不动手。
- **安全**：外部输入一律不可信（含 LLM 输出、第三方 API、报错文本）；参数化查询；不提交密钥；越权与不存在统一 404；鉴权在服务端。
- **性能**：先测量再优化；优化后同样方法复测；没超过噪声就回滚；N+1、无分页、无索引是常客。
- **调试**：先建能红的反馈回路（明确一条命令），再最小化复现，再列 3–5 个可证伪假设，一次改一个变量；清掉所有 `[DEBUG-*]`。
- **提交**：一个逻辑一件事；`<type>(<change-id>): <subject>`；提交前查密钥、跑测试；不把格式化与行为变更混在一起。

细节：TDD / 增量 / 调试 / API / 文档 / Git / 审查 / 迁移 → `reference/engineering.md`；安全 / 性能 / 可观测 / 五轮测试 → `reference/quality.md`。

---

## 6. 人机边界

- **默认直通**：上下衔接清楚、方案清楚、上一阶段已落盘 → 直接进下一阶段，不设"第 N 阶段必须停"的固定关卡。
- **只在四类澄清时停**：① 歧义（多套合理解法/平票）② 偏好（审美/取舍）③ 纠正（用户看过产物不满意）④ 把关（不可逆动作：提交、归档、破坏性变更）。
- **一次性问清**：同类疑问合成一条，解决后立即推进，不复述确认。
- **法定硬停点**：`<auto>false</auto>` 任务执行前；议题 ⑤ 人审门；违规处理协议。
- **暂缓项 = 议题**：不建 backlog / parking lot 同义词；统一登记进 `TOPICS.md` 或 STATE.md 一行，由 D-discovery 消化。

多 agent / 挂机协作（七类 agent、状态机、日志扫描 E1–E5、提交标准、降级策略）见 `reference/automation.md`。

---

## 7. 参考文件加载矩阵

> 每格标了"读哪节"；先 `grep -n '^##'` 看该文件的节标题，再 `read offset/limit`。

| 你要做的事 | 加载（节） |
|---|---|
| 守规矩 / 判断是否违规 | `RULES.md` 按 R 编号取节；速查用 `SYSTEM.md` |
| 路由 / 阶段细节 / 门禁构成 | `GO.md` 对应步骤；`prompts/<阶段>.md` |
| 写工件 | `templates/<工件>.md` |
| 恢复中断 / 反重复 | `RULES.md` R1；`templates/PROGRESS.md` |
| 少写代码 / 审查可删复杂度 / 债务台账 | `reference/ponytail.md` §2–3、§5–§7 |
| TDD、调试、API 设计、ADR、Git、废弃迁移 | `reference/engineering.md` §1–§7 |
| 安全、性能、可观测、五轮测试 | `reference/quality.md` §1–§5 |
| 质询、调研、PRD、UX、任务拆解 | `reference/planning.md` §1–§4 |
| 多 agent 自动化协作 | `reference/automation.md` |
| 想知道某个 multica/ponytail 能力落在哪 | `reference/coverage.md` |
| 技术栈 / UI 美学 / 反 slop / 测试金字塔 | `reference/tech-stacks.md` / `ui-aesthetics.md` / `ui-anti-patterns.md` / `test-pyramid.md` |

---

## 7.5 DSH 工具接线

| 场景 | 用什么 |
|---|---|
| 四类澄清（歧义/偏好/纠正/把关） | `ask_user_question`：一次一组、同类合并选项、给推荐项；解决后立刻推进 |
| 3-task 起跟踪进度 | `todo_write`（每完成一步立即更新） |
| 波次并行 / 独立评审 | `subagent`（后台默认，互不依赖的同消息并发） |
| ≥3 个同构任务扇出 | `workflow`（脚本编排，返回可 JSON 化结果） |
| 跨会话长目标 | `create_goal`（阻塞条件持续 3 轮才 blocked） |
| 读产物 | 先 `glob`/`grep` 定位，再 `read offset/limit`；不整读参考 |
| 验证 | `bash` 跑 verify 并贴真实输出；失败按 `reference/engineering.md` §2 |
| plan 模式下的方案 | `exit_plan_mode`（计划要决策完备） |

**DSH 没有常驻 agent**：`reference/automation.md` §1 的 ④④′⑤⑥ 是概念模型；默认走 §7 降级——每波次收尾由主 agent 亲自扫描/提交/对齐，前 3 阶段顺序做，不扇出。

---

## 8. 出口自检

- [ ] 阶段判定有依据（读了 STATE / 工件），没有凭印象跳步
- [ ] 本轮加载的 `reference/` ≤ 150 行（`templates/` 与用户明确要求全读除外）
- [ ] 产物已落盘到 `.specs/`，STATE.md 有状态行
- [ ] 实现类工作：verify 真跑过、输出已贴；diff 未越界；非平凡逻辑有测试
- [ ] 没有"看起来/应该/基本"式结论；没有把对话记忆当工件
- [ ] 该停的地方停了（歧义/偏好/纠正/把关），不该停的地方没问"要继续吗"
