# BMAD — code-kit 与 BMAD 技能族的集成映射

> BMAD 是技能族（bmad-* skills，已安装在 `~/.agents/skills/`），code-kit 是流程骨架（阶段 + 工件 + 门禁 + 硬规则）。
> **并存原则：code-kit 为主干，BMAD 为各阶段的深挖工具**。项目里已有 `.specs/` 时一律走 code-kit 命令，产物落规定位置；BMAD 技能按映射表在对应阶段按需借用。

自动化边界同样适用：BMAD 技能跑在 code-kit 项目里时，遵守 `AUTOMATION.md` 的澄清式人工介入与沉淀前置（阶段衔接清楚即直通，只在需要澄清时停）。

---

## 1. 映射总表

| code-kit 命令 | BMAD 技能 | 关系与取舍 |
|---|---|---|
| `D-discovery` | `bmad-discovery-loop` | **同构但权威只有一份**：code-kit 项目内一律走 `D-discovery`（主干前置阶段，kind: discovery；议题即 change，工件 `TOPICS.md`/`D-<子议题>.md`/`ROADMAP.md` 落 change 目录；**随步落盘 + 审核状态标**，R16.3/R16.5）。`bmad-discovery-loop` 技能是无 kit 环境（无 `.specs/`）的自由讨论版，其"过审前零写盘"是老语义——两版语义已分叉，**code-kit 项目内禁止用 bmad 版跑发现流程** |
| — | `bmad-forge-idea` / `bmad-brainstorming` / `bmad-deep-recon` | code-kit 无独立"锤炼想法"命令；想法未成型时先借这三个，产出喂给 `0-change` |
| `P-product` | `bmad-prd` / `bmad-product-brief` / `bmad-prfaq` | `P-product` 是 PR+原型**合一**（单文件 PRODUCT-DESIGN.html）；要纯文字 PRD 深挖时借 `bmad-prd`，逆向工作法借 `bmad-prfaq` |
| `2a-ui-design` | `bmad-ux` | code-kit 自带 28 页基线库 + tokens 硬约束，风格闭环更强；`bmad-ux` 的 DESIGN.md/EXPERIENCE.md 可作为 2a 的输入材料 |
| `2-design` | `bmad-architecture` / `bmad-spec` | 单 change 技术设计走 `2-design`；项目级架构梳理走 `A-architect`（≈ `bmad-architecture`）；把对话压缩成 spec 借 `bmad-spec` |
| `3-task` | `bmad-create-epics-and-stories` / `bmad-sprint-planning` | **接口不同**：code-kit 的原子任务（7 字段 + `<auto>` + 波次）是多 agent 执行接口；BMAD 拆出的 epic/story 若要进 `4-dev`，必须转写成 TASK.md 任务块 |
| `4-dev` | `bmad-build` / `bmad-build-auto` | 波次内逐 task 执行对应 `bmad-build`；`build-auto`（无人值守循环）≈ `<auto>true</auto>` 任务的自动连跑 |
| `5-test` | `bmad-qa-generate-e2e-tests` | `5-test` 从 AC 派生测试矩阵；E2E/API 自动化脚本生成借 bmad 版 |
| `6-review` | `bmad-code-review` / `bmad-review` | code-kit 双/三轮门禁为主干；`bmad-code-review` 可作跨模型 spot-check 轮的实现手段；`bmad-review` 评审任意非代码工件 |
| `7-integration` | `bmad-retrospective` | 归档 + LESSONS 提名 ≈ 复盘；可互引结论 |
| —（中途变更） | `bmad-correct-course` | 需求中途大改时评估影响面回到受影响阶段；code-kit 无对应命令，直接借 |
| `S-align` | —（无对应） | 五层对齐是 code-kit 独有；BMAD 侧漂移问题回到 `S-align` |
| `A-architect` / `A-evolve` | `bmad-architecture` | 项目级架构骨架 vs 架构深挖，可串用 |
| `I-intel-scan` | `bmad-project-context` | 老项目入场扫描 ≈ AGENTS.md 治理；一个出 CONTEXT.md，一个出 AGENTS.md，互补 |
| 专家团（R9/R13） | `bmad-party-mode` / 五个 agent 角色 | 门禁投票必须用 code-kit 12 角色体系（有投票格式 R13.3）；圆桌自由讨论可借 party-mode |
| — | `bmad-advanced-elicitation` | D-discovery ③ 步批判技法的加强版，结论拿不准时加一轮 |
| — | `bmad-walkthrough` | 澄清触发③（纠正：UI 人工过目）/ UAT 引导的辅助工具 |

## 2. 两套流程的阶段对照（心智模型）

```
code-kit:   0-change → 1-requirement → 2-design(2a) → 3-task → 4-dev → 5-test → 6-review → S-align → 7-integration
              │            │              │             │        │        │          │          │          │
BMAD:    forge-idea/   prd/product-   architecture/  create-   build/   qa-e2e   code-review/  correct-  retrospective
         brainstorm/   brief/prfaq    ux             epics/    build-            review        course
         deep-recon                                  sprint    auto
```

规模判定可互通：BMAD 的「琐碎→单会话→Epic→项目级」四档 ≈ code-kit 的「MVP 路径 → 单 change → 多 change」选择。

## 3. 使用守则

1. **工件以 code-kit 为准**：无论借了哪个 BMAD 技能，最终产物必须落 `.specs/` 规定位置（TASK.md / REQUIREMENT.md / …），否则后续阶段门禁（R2 Artifact Preflight）认不出来。
2. **门禁以 code-kit 为准**：阶段推进必须过 code-kit 的 R9 专家投票 + R13 自动化策略；BMAD 技能内部流程不能跳过这些门。
3. **澄清触发不豁免**：BMAD 技能产出触发 `AUTOMATION.md` §3.2 四类澄清（歧义/偏好/纠正/把关）之一时，照样停。
4. **入口互认**：装了 BMAD 技能的环境里，`/bmad` 是 BMAD 语义路由、`code-kit` skill 是 kit 路由；两边互指时按本表第 1 节跳转，不重复执行。
