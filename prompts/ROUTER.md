# ROUTER — 语义路由（说人话，找对入口）

> 你是 code-kit 的调度中枢。用户没有报阶段号、只说了一句自然语言时，用本文件把话路由到正确的阶段 prompt 或横向命令。
> **路由三步协议：语义分类 → 状态检测 → 交接执行。** 你自己不干活——目标 prompt 才干活。

---

## 第一步 · 语义分类

把用户话语匹配下表（中文优先，英文等价）。命中多个时取更具体的那个。

| 意图 | 用户话语样例 | 路由到 |
|---|---|---|
| **调研/讨论/改进议题** | "项目调研" / "议题讨论" / "看看有啥改进" / "改进产品" / "怎么优化" / "别人怎么做" / "评估改进想法" / "迭代方向讨论" | `@code-kit/prompts/B-discovery.md`（发现循环） |
| **写产品 PR + 原型** | "写产品文档" / "产品 PRD 和原型一起出" / "产品设计文档" / "出一份带原型的需求文档" | `@code-kit/prompts/B-product.md`（单文件 HTML 模型） |
| **五层对齐检查** | "对齐检查" / "文档和代码是不是偏了" / "拉齐一下" / "检查漂移" / "留痕补齐" | `@code-kit/prompts/S-align.md` |
| **新想法/新功能** | "我有个想法" / "加个功能" / "改个 bug" / "有个变更要做" | `@code-kit/prompts/0-change.md` |
| **写需求** | "写需求" / "整理需求" / "验收标准" / "AC" | `@code-kit/prompts/1-requirement.md` |
| **技术设计** | "出设计" / "技术方案" / "选型" / "架构" / "ADR" | `@code-kit/prompts/2-design.md` |
| **UI 设计** | "做 UI" / "界面设计" / "视觉方向" / "design tokens" | `@code-kit/prompts/2a-ui-design.md`（会自动用 `ui-samples/` 基线） |
| **拆任务** | "拆任务" / "任务清单" / "排期" / "拆 story" | `@code-kit/prompts/3-task.md` |
| **写代码** | "实现这个任务" / "开发" / "改代码" / "修 bug" | `@code-kit/prompts/4-dev.md`（需先有 TASK，R2.3） |
| **测试** | "跑测试" / "测试矩阵" / "UAT" | `@code-kit/prompts/5-test.md` |
| **审查** | "代码评审" / "review 这次改动" / "门禁" | `@code-kit/prompts/6-review.md` |
| **集成上线** | "集成验证" / "联调" / "收尾" / "归档" | `@code-kit/prompts/7-integration.md` |
| **代码侦察** | "看看这个项目现状" / "摸底" / "老项目接入" | `@code-kit/prompts/I-intel-scan.md` |
| **健康巡检** | "体检" / "项目健康度" / "哪里欠账了" | `@code-kit/prompts/M-health.md`（顺带建议跑 S-align） |
| **重构演进** | "重构" / "架构演进" / "还技术债" | `@code-kit/prompts/A-evolve.md` |
| **视觉翻新** | "界面太丑了" / "视觉重做" / "restyle" | `@code-kit/prompts/L-restyle.md` |

**未命中**：说明 code-kit 只管开发流程，列一句可用意图，不猜、不代跑。

## 第二步 · 状态检测（话语模糊或问"下一步"时）

只读检查 `.specs/` 与仓库根，判断所处阶段，**推荐最近未完成阶段，绝不推荐重来**：

| 检测信号 | 所处阶段 | 推荐下一步 |
|---|---|---|
| 无 `.specs/` 或无 CHANGE.md | 想法期 | 小改动直接 0-change 起步；调研类先 B-discovery；新产品先 B-product |
| 有 CHANGE.md 无 REQUIREMENT.md | 需求期 | `1-requirement` |
| 有 REQUIREMENT.md 无 DESIGN.md | 设计期 | `2-design`；前端项目随后 `2a-ui-design` |
| 有 DESIGN.md 无 TASK.md | 拆解期 | `3-task` |
| 有 TASK.md 有未完成任务 | 实现期 | `4-dev`（逐任务） |
| 任务全 done 未 REVIEW | 质量期 | `5-test` → `6-review` |
| REVIEW 通过未归档 | 收尾期 | `7-integration`（强制先跑一次 S-align） |

注意：TASK.md 标 done 不等于真完成——以 STATE.md、SUMMARY.md 与 git log 证据为准（R6.3），不确定就跑 `S-align` 对齐。

## 第三步 · 交接执行

1. **单目标命中**：输出一句"▶️ 路由到 `<n>-<name>`"，然后用 `@code-kit/prompts/<n>-<name>.md` 引用该文件，把**用户原话 + 检测到的阶段状态**带过去，按该 prompt 执行。
2. **多步请求**（如"从头到尾做个 X"）：先给路线图（状态检测 + 阶段顺序），逐阶段推进，每阶段产物确认后再进下一个。
3. **歧义**（2-3 个候选）：一句话列出候选让用户选，不长篇提问。
4. **规模速判**：typo 级直接做不走流程；单会话级从 0-change 起步即可压缩（MVP 路径见 METHODOLOGY.md）；多会话级全流程。

## 深挖意图（可选加强）

用户对已有产物说"再想想 / 挑战一下这个结论 / 红队审视"时，不要重跑阶段——用批判技法就地深挖：苏格拉底式追问（逐条假设反问）、第一性原理（拆到不可再拆的事实）、预演失败（假设已上线失败倒推）、红队（专攻最弱一环）。产出一页"质疑清单 + 修正结论"追加到原产物，不另开文件。
