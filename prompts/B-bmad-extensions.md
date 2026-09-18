# B-bmad-extensions — BMAD 能力在 code-kit 中的统一入口

> 这是融合层的调度 prompt，不是第二条研发流水线。先读 `BMAD-INTEGRATION.md`、`METHODOLOGY.md` 和 `RULES.md`，再按下表进入现有阶段或本文件的补充子流程。

## 1. 先判定子流程

| 用户意图 | 执行位置 | 产物/结果 |
|---|---|---|
| brief / PRD / PRFAQ | `P-product`；已有 change 则 `1-requirement` | `PRODUCT-DESIGN.html` / `REQUIREMENT.md` |
| UX / 用户旅程 / 无障碍 | `2a-ui-design` | `UI-DESIGN.md` |
| epic / story / sprint / readiness | `3-task` | `TASK.md` + `STATE.md` 状态与 readiness 记录 |
| e2e / QA | `5-test` | `TEST.md` 的 E2E 层和 UAT |
| walkthrough | `7-integration` | 人工验收清单，不替代 UAT |
| correct course / 重大变更 | `C-course-correction` | `COURSE-CORRECTION.md` |
| retrospective | `7-integration` 归档子流程 | `RETRO.md`，精选项进入 `.specs/LESSONS.md` |
| project context | `I-intel-scan`；写 AGENTS 时使用人工确认门 | `CONTEXT.md` / 经批准的 `AGENTS.md` |
| party / 多角色 / 深挖 | 当前阶段的专家团门禁 | 结构化意见、分歧、假设和决策 |
| 自动实现 | `4-dev` | 只对 `<auto>yes</auto>` task 自动推进，最多 3 轮 |

## 2. 产品文档补充

创建或更新产品文档时必须先收集：目标用户/场景、问题与价值、范围内外、成功指标及反指标、约束、利益相关者、风险、开放问题。PRFAQ 只作为“未来完成后的用户/利益相关者视角”补充，不另建平行需求源。所有 `[ASSUMPTION]` 必须进入开放问题，未确认前不升级为 AC。

## 3. Epic / Story / Sprint 补充

在 `TASK.md` 中使用：

```text
milestone: <epic 或里程碑>
task: <稳定 task id>
source: <REQUIREMENT/AC 引用>
depends_on: <task ids 或 none>
[auto]: yes | no | review
verify: <可执行验证>
done: <完成条件>
```

生成任务前检查：需求和设计完整、每个 task 可独立验证、依赖无环、范围与资源风险显式。`STATE.md` 是 sprint 状态真相源；不要引入第二个 status 文件。

## 4. Correct Course

重大变更必须只读盘点 CHANGE、REQUIREMENT、DESIGN、UI-DESIGN、TASK、TEST、REVIEW、当前 diff 和 STATE：

1. 说明触发问题与证据；
2. 列出各工件和代码的影响；
3. 提出直接调整、缩减范围或回退等候选；
4. 给出推荐方案、风险、工作量和时间影响；
5. 逐项展示变更建议，等待用户批准/编辑/跳过；
6. 用户批准后才更新上游工件，并重新跑受影响的下游门。

## 5. Retrospective / Walkthrough

归档前复盘：目标与实际、做得好、问题根因、可验证行动项、责任人与触发条件。不要把未经证据支持的意见写入长期 LESSONS。Walkthrough 面向人工审阅者说明变更目的、重点文件、风险和测试命令；它不修改代码，也不替代 review/test。

## 6. 安全与停止条件

保留 code-kit 的零外部依赖和安全边界。BMAD 上游需要脚本、安装、远程 handoff、提交或发布时，改为在当前项目内用现有 markdown/git 能力完成，或明确停下请求授权；不得为了兼容而执行未知脚本。所有自动循环最多 3 轮。
