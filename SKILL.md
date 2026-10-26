---
name: code-kit
description: code-kit AI 编程方法论总入口与语义路由。当用户提到 code-kit、@code-kit、GO.md，或任何 code-kit 流程语义时触发——立项/变更提案（change）、写需求/验收准则（requirement）、技术设计/选型（design）、UI 设计/design tokens/界面基线（ui-design）、拆任务/波次（task）、执行任务/断点恢复（dev）、测试矩阵/UAT（test）、代码审查（review）、集成归档（integration）、项目调研/议题发现前置阶段（discovery，议题即 change）、产品 PR+原型（product，前置阶段）、五层对齐/漂移检查（align）、多 agent 自动化/澄清式人工介入/挂机（automation）、BMAD 配合（bmad）、暂缓/议题（topics）、代码库巡检/换调性/入场扫描/架构梳理（横向命令）。路由到 kit 对应 prompt 文件并按其执行。
---

# code-kit — 语义路由与执行入口

你是 code-kit 流程的调度者。本 skill 是**薄路由**：kit 本体（prompts / templates / rules / 基线库）**随本 skill 目录自含**，是唯一事实源，**禁止把 kit 内容复制进对话记忆**——每次按下面的定位协议现读。

## 第一步 · 定位 kit 根目录

按优先级取第一个命中的路径作为 `<kit>`：

1. 当前项目根下的 `code-kit/` 目录（项目自带副本，用于项目级定制——须含 `GO.md`）
2. 本 skill 目录（即本仓库根目录，**正典本体**，随技能安装到任何 skills 目录都自含可跑）

验证：`<kit>/GO.md` 存在。两处都没有 → 报告 kit 本体缺失，重新安装本技能，停止。

> 正典唯一：本 skill 所在目录。kit 相关写读、diff、升级只对正典与项目内**新建**定制副本进行；旧分叉副本（升级遗留的过时版本）不引用、不同步、不采信，回落正典。

## 第二步 · 语义路由

用户话语命中下表哪条就走哪条（先具体后泛化；**新事物描述且无活跃 change 时一律先进 0-change**）：

| 意图（用户话语样例） | 读取并执行 |
|---|---|
| "立项" / "我想做 X" / "加个功能" / "修个 bug" | `<kit>/prompts/0-change.md` |
| "写需求" / "验收准则" / "AC" | `<kit>/prompts/1-requirement.md` |
| "技术设计" / "选栈" / "架构方案" | `<kit>/prompts/2-design.md` |
| "UI 设计" / "design tokens" / "界面风格" | `<kit>/prompts/2a-ui-design.md`（前端项目） |
| "拆任务" / "分解" / "排波次" | `<kit>/prompts/3-task.md` |
| "执行 T01" / "继续" / "恢复断点" | `<kit>/prompts/4-dev.md` |
| "写测试" / "UAT" / "测试矩阵" | `<kit>/prompts/5-test.md` |
| "审查" / "code review" | `<kit>/prompts/6-review.md` |
| "集成" / "验收上线" / "归档" | `<kit>/prompts/7-integration.md` |
| "调研" / "议题讨论" / "找改进" / "怎么优化" | `<kit>/prompts/D-discovery.md` |
| "产品文档" / "PR + 原型" | `<kit>/prompts/P-product.md` |
| "对齐检查" / "拉齐" / "文档代码偏了" | `<kit>/prompts/S-align.md` |
| "自动化" / "多 agent" / "挂机" / "哪里要人审" / "暂缓" / "下轮再说" | `<kit>/AUTOMATION.md` |
| "BMAD 怎么配合" / "用哪个技能" | `<kit>/BMAD.md` |
| 说不清 / 冷启动 / 纯 "code-kit" | `<kit>/GO.md`（完整路由协议，含 Token 预算红线） |

## 第三步 · 执行

1. **先读路由目标文件全文**，再按其指令执行——各 prompt 自带角色、输入、门禁、自检与触发下一步。
2. 阶段推进遵守 `<kit>/RULES.md` 硬规则（R1 token / R2 阶段门 / R13 专家投票 / R15 对齐 / R16 调研 / R17 自动化协作 / R18 人工关卡 / R19 阶梯律·最少实现）；违规时按 RULES 末尾协议输出并停下。
3. 工件一律落 `.specs/<change-id>/` 规定位置；跨会话状态写仓库根 `STATE.md`。
4. **DSH 环境**（可用 subagent / 后台 job 时）：3-task 拆解后按 `<kit>/AUTOMATION.md` 的七类 agent 分工执行——同波次任务用并行 subagent 派发、每任务 fresh context；日志扫描 / 自动提交 / 自动拉齐三件套可挂后台循环；纯对话环境按 AUTOMATION.md §7 降级为仪式。
5. **人工介入与推进**：上下衔接清楚、方案清楚的阶段直通推进（落盘即走，不问"要不要继续"）；唯一硬前置是上一阶段输出物已落盘（工件 + STATE.md）。人工只在需要澄清时停：歧义（4-5 套方案/平票）、偏好（UI 风格排版）、纠正（人看产物不满意→重生成）、把关（git 提交/破坏性变更落盘前）；法定硬停点另含 `<auto>false</auto>` 任务执行前、D-discovery 子议题 ⑤ 人审门（R18.2 全列以 RULES 为准）。

## 完成判据

- 产出该阶段 prompt「输出」一节要求的全部工件，且过其「自检」清单；
- 触发澄清时一次性问清（合并同类问题），解决后立即推进；
- 触发下一步时显式声明下一阶段文件路径。
