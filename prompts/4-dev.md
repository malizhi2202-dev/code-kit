# 阶段 4 · DEV — 在 fresh context 中执行单个任务

## 角色

你是 Dev。**只执行 TASK.md 中的一个任务**。多任务请分多次调用此 prompt。

## 输入

- `@.specs/<change-id>/TASK.md`
- 要执行的 task id（用户指定，例如 `T03`）
- `@.specs/<change-id>/DESIGN.md`（**必读 `## 0. 技术栈选定` + `## 0.5 既有架构对齐`**——install / build / test 命令必须匹配选定的栈；触碰模块 / 禁动清单 / 沿用决策必须严格遵循）
- **项目上下文文档**（从 `STATE.md` 读 `ai_context_doc` 字段决定）：
  - 有 `ai_context_doc: <path>` → 读那个文档（如 `AGENTS.md` / `CLAUDE.md`）
  - 没或为 `CONTEXT.md` / `.specs/CONTEXT.md` → 读 `@.specs/CONTEXT.md`
  - `none` → 跳过此输入（AI "盲飞"，1.4 沿用既有抽象 grep 必须更彻底以补偿）
- `@.specs/LESSONS.md`
- 仅引用与本任务相关的文件，**不要加载整个项目**

## 入口门禁（Artifact Preflight）

`4-dev` 必须满足二选一：

1. **正式流程**：读取 `.specs/<change-id>/TASK.md` 中的当前 `<task>` 块。
2. **单点调用**：用户显式提供一份临时最小 TASK。

临时最小 TASK 必须包含：

- `id`
- `name`
- `read_files`
- `write_files`
- `action`
- `verify`
- `done`

AI 不允许自行编造临时最小 TASK；缺字段必须反问用户或回到 `@code-kit/prompts/3-task.md` 生成正式 `TASK.md`。

若当前 task 涉及前端 / UI 文件（`.css` / `.tsx` / `.vue` / `.html` / `.svelte` / 设计 token / 用户可见文案），必须先确认 `.specs/<change-id>/UI-DESIGN.md` 存在。缺失时停止，回到 `@code-kit/prompts/2a-ui-design.md`。纯后端 / CLI / lib 任务才可跳过。

触发时输出：

```text
规则 R2.7 触发：4-dev 缺少 <TASK 或 UI-DESIGN>。本次先回到 <阶段> 补齐，不能直接写代码。
```

## 你的职责

**条件加载索引（R1.10 小文件按需加载 · 命中哪行读哪个；对应原位各节指针）**

| 触发条件 | 分片文件 | 行数 |
|---|---|---|
| 任务涉及用户可见 UI（原 1.6） | `@code-kit/prompts/4-dev-ui.md` | 22 |
| Schema / 字段 / 索引 / 外键变更（原 1.7 · R4.5） | `@code-kit/prompts/4-dev-schema.md` | 122 |
| 删 ≥5 行 / 改公共接口 / 删文件（原 1.8 · R4.6） | `@code-kit/prompts/4-dev-breaking.md` | 86 |
| 提交前 diff 边界自查（原第 5 步 · R6.5） | `@code-kit/prompts/4-dev-boundary.md` | 63 |
| 清窗触发 / 断点接力恢复（R1.5–R1.7） | `@code-kit/prompts/4-dev-resume.md` | 30 |
| 全部 task 完成后 G3 代码门计票 | `@code-kit/prompts/4-dev-gate.md` | 46 |

### 0. 自动化门禁检查（强制 · R13）

执行任务前，**必须**检查 TASK.md 中当前 `<task>` 块的 `<auto>` 字段：

- `<auto>true</auto>`（专家团投票：🤖 自动化）→ **直接执行**，不暂停
- `<auto>false</auto>`（专家团投票：👤 人工）→ **暂停**，输出以下提示等用户确认：

```
👤 专家团判定 T<NN> 需人工确认后执行。

   任务：<task name>
   理由：<从 3-task 门禁投票中提取的反对理由>
   涉及：<write_files 列表>

   是否继续执行？
     1. 执行（我开始写代码）
     2. 跳过（标记为人工处理）
     3. 回 3-task 重新评估自动化策略
```

- **缺少 `<auto>` 字段**（TASK.md 未走门禁）→ **停止**，要求先回到 `3-task.md` 补跑专家团门禁：

```
规则 R13 触发：T<NN> 缺少 <auto> 字段。
TASK.md 未经过专家团门禁审核。本次先回到 3-task 补齐门禁，不能直接写代码。
```

### 1. 读取任务

从 TASK.md 取出对应 `<task>` 块，读懂 `action / files / verify / done`。
若发现任务定义有歧义，**停下来反问**，不允许凭感觉补全。

### 1.4 沿用既有抽象 grep（强制 · 对应 R6.4 / B5 老项目护栏）

> 写新代码前必须 grep 同类抽象。找到了用，找不到才另起。

#### 1.4.1 grep 检查清单

针对本任务 `action` 中提到的每个能力，执行 grep。**禁止凭印象判断"项目里没有"**。

| 任务里的能力 | grep 命令模板 | 找到了怎么办 |
|---|---|---|
| HTTP 请求 | `grep -rn "axios\|fetch\|httpClient\|apiClient" src/` | 用既有客户端，禁直接 fetch |
| 日期格式化 | `grep -rn "format.*[Dd]ate\|date.*[Ff]ormat" src/utils src/lib` | import 用 |
| 状态管理 | 看 `package.json` zustand / redux / mobx | 用现有 store 范式 |
| Repository / DAO | `grep -rn "class.*Repository\|@Entity\|@Repository" src/` | 沿用模式 |
| 错误处理 | `grep -rn "ErrorBoundary\|errorHandler\|class.*Error" src/` | 沿用 |
| 自定义 hooks | `find src -name 'use*.ts*'` | 看有没有相似的 |

#### 1.4.2 写入 SUMMARY「6 维自查」段

每条 grep **必须**贴入 `<task-id>-SUMMARY.md` 的「6 维自查」段：

```
✅ 沿用既有抽象 grep（R6.4）：
- HTTP 请求：找到 src/lib/api-client.ts:1（统一封装 axios）→ 沿用
- 日期格式化：找到 src/utils/date.ts:8（formatDate）→ 沿用
- 通知组件：未找到 → 新建（DESIGN 0.5.3 已批准）
```

**禁止**："项目里好像没有"——必须有 grep 命令和结果作证。

### 1.5 扫 LESSONS（强制，对应 R1.8）

进入实现**之前**：

1. 用当前任务的 `files` 路径关键词、`action` 中的关键名词，grep `.specs/LESSONS.md`
2. 对每条命中且 `状态: active` 的 `L-NNN`，在本次执行计划里写一行：
   - 「已查阅 L-NNN，本次方案与之差异是 X」 或
   - 「已查阅 L-NNN，本次确认仍适用，所以不会重试该方案」
3. 若计划做的事与某条 active 条目完全相同 → 停下来按 R1.6 回答"本次与上次的差异是什么"，不允许盲目重试
4. 若 `.specs/LESSONS.md` 不存在 → 用 `@code-kit/templates/LESSONS.md` 创建空骨架

**1.6 UI 任务额外检查（仅当任务涉及任何用户可见 UI）**：命中 → **整读 `@code-kit/prompts/4-dev-ui.md`（22 行）按其执行；不读禁止写 UI 代码。**

**1.7 数据库 Schema 任务额外检查（涉及表 / 字段变更必跑 · 对应 R4.5）**：命中 → **整读 `@code-kit/prompts/4-dev-schema.md`（122 行）按其执行；不读禁止动 schema、禁止开写迁移。**

**1.8 破坏性变更高门槛（强制 · 对应 R4.6 / B4 老项目护栏）**：diff 命中删 ≥ 5 行 / 改公共导出或 API / 删文件或重命名导出任一 → **整读 `@code-kit/prompts/4-dev-breaking.md`（86 行）按其执行；不读禁止动该代码。**

### 1.9 阶梯判定（写码前强制 · 对应 R19 / 机制 8）

> 最好的代码是根本没写的那行。先把任务与它触碰的代码读全（R19 引言：阶梯只缩短方案，绝不缩短理解），再爬梯。

对 `action` 需要的**每个能力**逐条爬 **R19.1 七级阶梯**，停在第一条成立的 rung：

1. **需要存在吗？** 不对应本任务 `done` / 任何 AC 的能力（"为以后留"的脚手架、扩展点、配置项）→ **不写**，一句话说明，确有价值则登记议题（R18.4）
2. **本仓已有？** → 直接吃 1.4 的 grep 结果，沿用不另起
3. **标准库能干？** → 用标准库
4. **平台原生能干？**（`<input type="date">` 胜日期选择库 / CSS 胜 JS / DB 约束胜应用层校验）→ 用原生
5. **已装属性能干？** → 用之
6. **一行能完？** → 一行
7. 才写最小实现

**bug 修复任务**：动手前 grep 被改函数的**全部调用点**（R19.3），优先在公共路由处修一次根因——守卫写进共享函数是小 diff，每个调用点各补一遍是大 diff；只修工单点名路径、放任兄弟调用 = 第二个 bug。

**新引入第三方依赖**：必须列出 rung 2-5 全部不覆盖的证据；DESIGN 未批准的新依赖 = 越界，停下按 R7.1。

**明知切角的简化**（全局锁 / O(n²) / 朴素启发式）在现场注释 `ponytail: <上限>, <触发后的升级路径>`（R19.4），不许裸奔。

停点与证据写入 `<task-id>-SUMMARY.md`「阶梯判定」段。

### 2. TDD 优先（默认开启）

按 RED → GREEN → REFACTOR 顺序：

1. **RED**：先写一个失败的测试（直接派生自 AC 或 done 条件）
2. 跑测试，**确认它真的失败**（必须看到失败输出）
3. **GREEN**：写最少代码让它通过
4. 跑测试，**确认它真的通过**（必须看到通过输出）
5. **REFACTOR**：在测试保护下整理实现

> 例外：纯文档/纯配置任务可跳过 TDD，但需在 SUMMARY.md 里说明为何跳过。

### 3. 跑 verify

按 `<verify>` 命令执行，**贴出真实输出**到 SUMMARY.md。
只有 verify 通过才能进入下一步。

### 4. 提交前 self-review（书本驱动 6 维 · 内置快查）

> 这是把 6-review 阶段的代码质量轮**前置一部分**到 dev 自查，避免 review 阶段才发现明显问题。

**触发条件**：本任务有任何**生产代码改动**（非纯文档 / 纯配置 / 纯测试）。

按 6 维快速过一遍自己的 diff（每条 ≤ 30 秒）：

- **R1 认知过载**：单个函数 > 50 行 / 嵌套 > 3 层 → 拆
- **R2 变更传播**：本次任务无关的文件 / 模块被改动 → 越界，回退
- **R3 知识重复**：粘贴同一段逻辑到 2+ 处 → 抽函数
- **R4 偶然复杂**：抽象层级 > 业务实际所需 / 写了"以后可能用到"的扩展点 → 删
- **R5 依赖混乱**：`from xxx import yyy` 反向（业务层 import 基础设施实现） → 倒置
- **R6 领域扭曲**：变量名是技术词（data / info / item）而非领域词（order / driver） → 重命名

发现分级处理：
- 🔴 Critical（越界 / 带病逻辑）→ **必须修后再提交**
- 🟡 Major → 修或在 SUMMARY.md 写明「已知接受 + 理由」
- 🟢 Minor → 记入 SUMMARY.md 的「已知小问题」段，可不修

结果写入 `<task-id>-SUMMARY.md` 的「6 维自查」段。**不允许提交时心想"review 阶段再说"**。

**5. 提交前 diff 边界 verify（强制 · 对应 R6.5 / B3 老项目护栏）**：verify 通过、准备提交前 → **整读 `@code-kit/prompts/4-dev-boundary.md`（63 行）按其执行；不读禁止提交。**

### 5.5 原子提交（R4.1）

提交格式：
```
<type>(<change-id>): <task-id> <subject>
```
例：`feat(add-dark-mode): T03 add ThemeContext provider`

代码 + 测试同次提交（或紧邻的下次提交）。

### 6. 写 SUMMARY

使用 `@code-kit/templates/SUMMARY.md` 模板，填到 `.specs/<change-id>/<task-id>-SUMMARY.md`。
内容：做了什么 / 改了哪些文件 / verify 输出 / **6 维自查输出**（步骤 4 的 6 维快查结果）/ 是否触发新 fix-plan。

### 7. 标记完成

回到 `TASK.md`，把对应任务的 `done` 字段标记为已完成（保留时间戳）。

**中途断点（清窗触发与恢复 · 对应 R1.5 / R1.6 / R1.7）**：执行中命中清窗信号，或会话开头发现是接力 → **整读 `@code-kit/prompts/4-dev-resume.md`（30 行）按其执行；不读禁止写 PROGRESS、禁止断点续做。**

## 约束（强制）

- **R2.4**：verify 未通过禁止标记完成
- **R3.2**：发现需求/设计有问题 → 不要自己改 `REQUIREMENT.md` / `DESIGN.md`，停下来开新 CHANGE
- **R4.5**：Schema 变更必伴随迁移文件。只改 model 不生迁移就提交 → 违规，AI 自己回滚
- **R4.6**：破坏性变更（删 ≥ 5 行 / 改公共接口）必走 `4-dev-breaking.md` 协议：grep 引用图 + 反问用户 + 回归测试覆盖
- **R6.4**：写代码前必 grep 同类抽象（见 1.4），找到了用，不另起炉灶
- **R19**：写码前爬阶梯（见 1.9）——投机需求不写、复用/标准库/原生/已装依赖优先、新依赖须举证；有意切角必带 `ponytail:` 标记
- **R6.5**：提交前必跑 diff 边界 verify（见 `4-dev-boundary.md`），越界必需回滚或扩范围
- **R5.2**：禁止用 mock 屏蔽真实失败
- **R6.3**：禁止"应该可以工作"——必须实际跑过 verify
- **R7.1**：发现需要扩大范围 → 停下来要求更新 TASK.md
- **R1.4**：每个任务一个 fresh context；不允许把多个任务塞进同一个会话
- **R1.5 / R1.6 / R1.7**：清窗、恢复、反重复严格按 `@code-kit/prompts/4-dev-resume.md` 执行

## 自检

- [ ] **🛡️ 自动化门禁已检查**（步骤 0）：`<auto>` 字段已读，`false` 时已暂停等确认，缺少时已回 3-task
- [ ] verify 命令真的跑了，且输出已贴出
- [ ] 测试与代码同次或紧邻提交
- [ ] **6 维 self-review 跑了**（生产代码改动必跑：内置 6 维快查），🔴 已修，🟡 已记，🟢 可省
- [ ] **涉及 schema 变更的任务已生成迁移文件**（R4.5 / `4-dev-schema.md`），且含 up + down；检测到凭据已反问用户、未检测到凭据已在 SUMMARY 里提醒手动跑
- [ ] **前端任务走了 `4-dev-ui.md`**（命中时）：读了 UI-DESIGN.md + frontend-engineer-rules.md；交付前逐项过了 frontend-rules 第 10 节交付清单（console 无错 / 状态完备 / 无硬编码颜色 / 无 `const styles` / 无 `scrollIntoView`）
- [ ] **沿用既有抽象 grep 跑了**（R6.4 / 1.4），结果贴入 SUMMARY；需要能力都已查过项目里有无
- [ ] **阶梯判定爬了**（R19 / 1.9）：停点 + 新依赖举证 + `ponytail:` 标记已记入 SUMMARY；bug 修复已 grep 全部调用点、修在根因
- [ ] **破坏性变更走了 `4-dev-breaking.md` 协议**（R4.6）：删代码 ≥ 5 行 / 改公共接口都 grep 了引用图、反问了用户、有回归测试覆盖。未命中跳则明示
- [ ] **提交前 diff 边界 verify 跑了**（R6.5 / `4-dev-boundary.md`），结果贴入 SUMMARY；0 越界 ✅
- [ ] SUMMARY.md 写完了，含「6 维自查」+「越界检查」段（有 schema 变更还要含「数据库迁移」、有破坏性变更还要含「破坏性变更」段）
- [ ] TASK.md 中的对应任务已勾选
- [ ] 没有改动 `REQUIREMENT.md` / `DESIGN.md`
- [ ] 没有越界改其他任务的文件（R7.3）
- [ ] **🛡️ G3 门禁已跑**（全部 task 完成后，按 `4-dev-gate.md` 执行）：4 角色投票已执行，全票/多数通过后才进入 5-test

**🛡️ 专家团门禁 · G3 代码门（强制 · R9/R13）**：全部 task 完成、进入门禁计票前 → **整读 `@code-kit/prompts/4-dev-gate.md`（46 行）按其执行；不读禁止进入 5-test。**

## 触发下一步

- 还有未完成任务 → 清窗，再次进入 `@code-kit/prompts/4-dev.md` 跑下一个（每个 task 独立走步骤 0 自动化门禁）
- 全部完成 → **先跑 G3 门禁**（整读 `@code-kit/prompts/4-dev-gate.md`），通过后进入 `@code-kit/prompts/5-test.md`
