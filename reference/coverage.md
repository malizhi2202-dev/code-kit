# 能力覆盖表（整合溯源）

> 本仓库在原有 code-kit 方法论（`GO.md` / `RULES.md` / `METHODOLOGY.md` / `prompts/` / `templates/` / `reference/` / `ui-samples/`）之上，吸收了两部分上游能力：
> 1. **multica starterkit 的 30 个技能**（收 29 个）；
> 2. **ponytail**（DietrichGebert/ponytail，MIT）的完整能力。
>
> 目的：能力不丢、可追溯、避免重复维护。运行时不必读本表，只在"这个能力从哪来/有没有丢"时查。

---

## 一、multica starterkit · 14 个流程技能

| 原技能 | 能力 | 落在 |
|---|---|---|
| `kit-entry` | 统一入口、意图路由、工件预检、加载预算、老项目入场、路由声明 | `SKILL.md`；`GO.md` |
| `kit-rules` | 系统硬规则（原 R1–R16 + 新增 R17/R18/R19） | `RULES.md`（R1–R8）+ `RULES-FLOW.md`（R9–R19）；`SYSTEM.md` 精简版 |
| `kit-roles` | 12 角色 + 8 门 + 投票格式 | `go-gates.md` 门禁总览；`prompts/R-*.md` |
| `kit-automation` | 七类 agent、状态机、日志扫描、提交标准、降级 | `AUTOMATION.md`（主干协议）；`RULES-FLOW.md` R17/R18 |
| `kit-horizontal` | D-discovery / P-product / S-align / M-health / I-intel-scan / A-architect / A-evolve / L-restyle | `prompts/` 对应命令；`reference/planning.md` §3（P-product） |
| `kit-0-change` | change-id、架构预检、调性预选、想法压测 | `prompts/0-change.md` |
| `kit-1-requirement` | 用户故事、Given/When/Then、v1/v2/out、域语言 | `prompts/1-requirement.md` |
| `kit-2-design` | 技术栈、既有架构对齐、决策/ADR/风险/§9 | `prompts/2-design.md`；`reference/tech-stacks.md` |
| `kit-2a-ui-design` | 调性、视觉语汇对齐、v0、tokens、占位符、anti-slop | `prompts/2a-ui-design.md`；`reference/ui-*.md`；`ui-samples/` |
| `kit-3-task` | 原子任务、read/write 边界、verify、波次、`<auto>` | `prompts/3-task.md`；`templates/TASK.md` |
| `kit-4-dev` | 自动化门禁、复用 grep、LESSONS、schema 迁移、破坏性协议、TDD、6 维、diff 边界、提交、PROGRESS | `prompts/4-dev.md`；`RULES.md` R4/R6；`templates/` |
| `kit-5-test` | 五轮金字塔、AC 矩阵、T1–T6、量化判定 | `prompts/5-test.md`；`reference/test-pyramid.md`；`reference/quality.md` §4 |
| `kit-6-review` | spec/质量/UI 三轮、6 维衰退、修复任务 | `prompts/6-review.md` |
| `kit-7-integration` | S-align 前置、全量自动化、UAT、LESSONS 提名、归档 | `prompts/7-integration.md`；`templates/ALIGNMENT.md` |

## 二、multica starterkit · 16 个工程/调研/规划技能（收 15）

| 原技能 | 能力 | 落在 |
|---|---|---|
| `test-driven-development` | RED/GREEN/REFACTOR、Prove-It、金字塔、DAMP、mock 分级 | `reference/engineering.md` §1 |
| `diagnosing-bugs` | 反馈回路、最小化、可证伪假设、插桩、回归、清理；测试/构建/运行时三分诊 | `reference/engineering.md` §2 |
| `api-and-interface-design` | Hyrum、One-Version、Contract First、错误语义、分页、幂等键、深模块/接缝 | `reference/engineering.md` §3 |
| `documentation-and-adrs` | ADR 模板与生命周期、注释、API 文档、README、changelog、agent 文档 | `reference/engineering.md` §4 |
| `git-workflow-and-versioning` | 主干、原子提交、信息、规模、worktree、提交卫生、SemVer、changelog、冲突、发布清单 | `reference/engineering.md` §5 |
| `deprecation-and-migration` | 废弃决策、建议/强制、Strangler/Adapter/flag、expand-contract、僵尸代码 | `reference/engineering.md` §7 |
| `incremental-implementation` | 薄垂直/契约优先/风险优先切片、实现→测试→验证→提交、flag、可回滚、范围纪律 | `prompts/4-dev.md`；`reference/planning.md` §4 |
| `code-review` | 固定基准点、双轴审查、Fowler 坏味道、约束契约、存疑复核、走查 | `reference/engineering.md` §6；`prompts/6-review.md` |
| `security-and-hardening` | 威胁建模、STRIDE、三档纪律、OWASP 模式、SSRF、供应链、隐私、LLM 安全 | `reference/quality.md` §1 |
| `performance-optimization` | 测量工作流、CWV、N+1/索引/连接池/缓存、复测与回滚、预算 | `reference/quality.md` §2 |
| `research` | 一手来源、深度侦察三式、研究防火墙、时效与引用、源码驱动开发 | `reference/planning.md` §2 |
| `grilling` | 设计树访谈、前沿、事实自取、决策交用户；一次一问/圆桌/深挖/教用户/生成向导/头脑风暴三立场 | `reference/planning.md` §1 |
| `bmad-prd` | 脑暴→赌注→模式、引导不代写、关切扫描、PRFAQ、审阅门、收尾 | `reference/planning.md` §3.1 |
| `bmad-ux` | DESIGN/EXPERIENCE 双契约、捕获不代写、表面闭合、评审镜头、一次性原型 | `reference/planning.md` §3.2–3.3 |
| `planning-and-task-breakdown` | 依赖图、垂直/契约优先/风险优先切片、任务结构/规模、检查点、并行、不覆盖未完成计划、issue/规格/分诊/决策地图 | `reference/planning.md` §4 |
| `scaffold-exercises` | 课程仓库的练习脚手架、Obsidian 库、shoehorn 迁移、lint 规则 | **未收**：绑定特定课程仓库与内部 CLI，无通用能力；同类需求走 `0-change` + 普通脚手架任务 |

## 三、ponytail（DietrichGebert/ponytail, MIT）

| 能力 | 落在 |
|---|---|
| 最懒资深工程师哲学、持久化、等级 lite/full/ultra | `reference/ponytail.md` §0–3；`RULES-FLOW.md` R19 |
| 阶梯（存在吗 → 仓库有吗 → 标准库 → 平台 → 已装依赖 → 一行 → 最少代码） | `reference/ponytail.md` §2；`SKILL.md` §3 |
| 范围定义（调用方/测试/夹具/配置）、根因修复 | `reference/ponytail.md` §1 |
| 绝不省的清单、`ponytail:` 注释（历史 `shortcut:` 同识别；执行钩子：`prompts/4-dev.md` §1.9、`templates/SUMMARY.md`「阶梯判定」、`prompts/M-health.md` §2.5 收割） | `reference/ponytail.md` §3；`RULES-FLOW.md` R19 |
| 输出纪律（代码优先、结尾三行） | `reference/ponytail.md` §4 |
| diff 审查（上游 `/ponytail-review`，四段发现、三组排序、Verdict/Lean/Not checked） | `reference/ponytail.md` §5；`prompts/6-review.md` 可追加此视角 |
| 全仓审计（上游 `/ponytail-audit`，排序、上限 20） | `reference/ponytail.md` §6；`prompts/M-health.md` |
| 捷径台账（上游 `/ponytail-debt`，no-trigger） | `reference/ponytail.md` §7 |
| 基准与诚实边界（不编 per-repo 节省） | `RULES-FLOW.md` R19.6；`reference/ponytail.md` §3 |
| 与 code-kit 各阶段的接线 | `reference/ponytail.md` §8 |
| 判定例子（日期选择器/邮箱校验/防抖） | `reference/ponytail.md` §9 |

## 四、未收的 multica 内置技能（starterkit 之外，产品绑定）

| 技能 | 原因 |
|---|---|
| `multica-onboarding` | multica 产品的新手引导，绑定其工作区/issue/agent 模型 |
| `multica-platform` | multica 平台操作说明 |
| `multica-working-on-issues`（legacy） | 绑定 multica issue 流 |
| `incident-response`（示例插件） | 示例插件专用，非通用；通用事故处理已由 `reference/engineering.md` §2（调试/恢复）覆盖 |

> 若未来要做 multica 平台类工作，应回到 multica 仓库读原技能，不要从本仓库反推。

## 五、保留的原技能能力（引用但未逐字复制）

- **BMAD 的脚本/配置体系**（`resolve_customization.py`、`memlog.py`、`render_skill.py`、`uv run`）：属于 BMAD 安装运行时，不是方法论；本仓库只吸收流程思想（脑暴→赌注→模式、捕获不代写、双契约、评审门），不复制依赖。需要原样跑 BMAD 时回原仓库。
- **multica 的模板文件包**：`templates/` 已提供等价骨架；完整模板以项目实际约定为准，不硬绑某仓库路径。

## 六、维护约定

1. 新增能力先判断落在哪个文件（阶段 → `prompts/`；工程底座 → `reference/`；规则 → `RULES.md` / `RULES-FLOW.md`）；不新建平行概念（与 R18.4 一致）。
2. 上游升级时只改对应文件的一节，不在多处复制。
3. 本表是溯源索引，不是运行时必读。
