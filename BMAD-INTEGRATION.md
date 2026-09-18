# BMAD-INTEGRATION — code-kit 的 BMAD 能力融合层

> 本文件不是 BMAD 的副本，而是把 BMAD 的决策质量、产品规划、变更治理和交付运营能力，映射到 code-kit 的工件、门禁和状态模型中。
>
> **核心原则**：code-kit 的 `.specs/<change-id>/`、阶段门禁、token 预算、零外部依赖和安全边界继续有效；BMAD 的大议题/子议题发现流程是兼容锁定流程，不得压缩、改序或自动落盘。

## 一、能力合并策略

| BMAD 能力 | code-kit 落点 | 合并方式 |
|---|---|---|
| 产品 brief / PRD / PRFAQ | `P-product` → `PRODUCT-DESIGN.html`，或 `1-requirement` | 补充产品定位、用户价值、成功/反指标、利益相关者和 FAQ；技术实现仍留在 DESIGN |
| 大议题 / 子议题发现 | `D-discovery` | 保留 BMAD 的三轮、八步大议题顺序、逐子议题、逐轮人工确认；事实依据与 code-kit discovery 产物合并 |
| 头脑风暴 / 深度 elicitation / idea forge | `D-discovery`，或已有工件的“深挖意图” | 只补问题探索，不另开重复文档；结论未经确认不得成为需求 |
| UX / UI / 无障碍 | `2a-ui-design` | 将用户旅程、可用性、无障碍和界面 token 合并到 UI-DESIGN |
| 架构 / ADR / 技术选型 | `A-architect` + `2-design` | 项目级决策先进 ARCHITECTURE，change 级决策进 DESIGN；避免两处重复维护 |
| Epics / Stories / Sprint Planning | `3-task` + `TASK.md` + `STATE.md` | Epic 映射为 milestone，story 映射为 task；新增 readiness gate、状态汇总和依赖校验 |
| Build / Dev / QA E2E | `4-dev` + `5-test` | 完整实现、测试先于实现派生自 AC；E2E 作为 TEST 的独立验证层 |
| Code Review / Walkthrough | `6-review` + `7-integration` | review 只产报告；walkthrough 作为人工验收说明，不代替门禁 |
| Correct Course | `S-align` + 新 change / `prompts/C-course-correction.md` | 先分析 PRD/需求/架构/UI/TASK 的影响，再由用户批准是否更新工件 |
| Retrospective / Lessons | `7-integration` + `.specs/LESSONS.md` | 归档前复盘，验证行动项；可沉淀为 LESSONS，不写成空泛总结 |
| Project Context | `I-intel-scan` + `AGENTS.md`/`CONTEXT.md` | 扫描和采用上下文，写入前必须展示并获用户批准 |
| Party / 多角色 / 高级 elicitation | 现有专家团门禁 | 不复制 persona；按阶段临时组专家，输出结构化意见与分歧 |
| Build-auto | `4-dev` 的自动模式 | 仅对 task 的 `<auto>` 投票通过项启用，最多 3 轮，超限人工介入 |

## 二、统一工件映射

BMAD 术语不新增第二套真相源：

- Product brief / PRD → `CHANGE.md`、`REQUIREMENT.md`、可选 `PRODUCT-DESIGN.html`
- Epic / Story → `TASK.md` 的 `milestone` / `task` 字段
- Architecture / UX → `ARCHITECTURE.md`、`DESIGN.md`、`UI-DESIGN.md`
- Sprint status → 根 `STATE.md` 的 change/task 状态与 `.specs/<id>/PROGRESS.md`
- Review / QA → `REVIEW.md`、`TEST.md`、`UAT.md`
- Change proposal → `.specs/<id>/COURSE-CORRECTION.md`
- Retrospective → `.specs/<id>/RETRO.md`，筛选后的长期经验进入 `.specs/LESSONS.md`

若发现 BMAD 风格旧工件（如 PRD、epics、sprint-status），先做只读映射；不得默默覆盖或删除，冲突走 `S-align`。

## 三、统一质量门

1. **探索门**：大议题/子议题必须按 `D-discovery` 的确认锁执行。
2. **产品门**：PRD/brief/PRFAQ 的用户价值、范围、成功指标和未决问题明确后，才能生成 REQUIREMENT。
3. **设计门**：架构、UX、技术、数据、安全和可运营性决策记录在 DESIGN/ARCHITECTURE。
4. **实现就绪门**：每个 task 有输入、边界、验证、完成条件；依赖和并行标记清楚。
5. **交付门**：AC、测试、review、对齐、UAT 和归档记录一致。

门可以在一轮对话内连续完成，但不能缺工件、跳过人工确认或把未确认假设写成结论。

## 四、路由契约

所有自然语言先经过 `ROUTER.md`，再做 Artifact Preflight。BMAD 特有能力统一路由至 `prompts/B-bmad-extensions.md`；该 prompt 只负责选择子流程和产物，不绕过 code-kit 阶段。
