# 横向命令 · A-evolve — 把 change 期间的架构沉淀同步到项目级文档

> **触发方式**：`@code-kit/GO.md` + `同步架构 / 整理沉淀 / sediment / evolve / 架构演进 / 同步 CONTEXT`
> 不属于任何 change，不写 CHANGE.md / REQUIREMENT.md。直接产出沉淀同步报告 + patch CONTEXT.md（+ ARCHITECTURE.md如存在）。

## 角色

Architect-Evolver。**只读 archived change 的 DESIGN.md § 9，聚合分类后让用户 review，批准的才 patch 到项目级文档。不动业务代码**。

## 与 A-architect 的边界

| 工作流 | 干什么 | 何时跑 | 改 ARCHITECTURE.md 哪段 |
|---|---|---|---|
| **A-evolve**（本文）| 把 change 级沉淀**单点 append** 到项目级文档 | 每月 / 每季 批量 | 仅 append ADR 列表 / 跨模块契约段 / 修订历史 |
| **A-architect** | 重写 / 大改 ARCHITECTURE 全篇 | 首次建立 / 重大重构 / ADR 重审 | 任何段都可改 |

如果 A-evolve 过程中发现需要“改架构”（比如某条 ADR 应该 deprecated / 依赖规则该改）→ **停下来，提示用户跑 A-architect**，不在本工作流里动。

---

## 触发场景

- **每月 / 每季度**：积累了若干 change 后批量同步一次，避免 CONTEXT 滞后
- **里程碑后**：版本发布或大功能完成后，把这一阶段的沉淀凝固到项目层
- **CONTEXT 失准信号**：4-dev 阶段连续多次发现 AI 没沿用既有抽象（说明索引漏了）
- **`STATE.md` `last_evolve_at` 字段为空 或 距今 > 60 天**：GO.md 在路由时会主动提示
- **M-health 冗余巡检留的尾巴**：若 `.specs/CONTEXT.md`「清理窗口专列」或「技术债」段有标记，A-evolve 扫描时**主动把对应的「既有抽象索引」条目拎出来让用户确认是否删**（闭环 M-health 步骤 5 的"下次同步主动提醒"）

## 输入

- `@STATE.md`（仓库根；读 `last_evolve_at` 字段，决定从哪个时间点之后的 change 开始扫）
- `@.specs/CONTEXT.md`（当前快照，patch 目标 · rules 层）
- `@.specs/ARCHITECTURE.md`（如存在，则同时是 patch 目标 · structure 层）
- `@.specs/archive/<YYYY-MM-DD>-<change-id>/DESIGN.md`（已归档 change 的设计文档 · 只读 § 9 段）
- `@.specs/<active-id>/DESIGN.md`（当前活跃 change 也算，但仅在 4-dev 完成 / 7-integration 之后才纳入 · 否则跳过）
- `@.specs/CHANGELOG.md`（看 change 元信息，做 cross-check）

---

## 你的职责

### 步骤 1 · 确定扫描范围

读 `STATE.md`：

- 有 `last_evolve_at: <YYYY-MM-DD>` 字段 → 只扫该日期**之后**归档的 change
- 无字段（首次跑）→ 扫所有 `.specs/archive/*` 的 DESIGN.md
- 有 `last_evolve_promoted: [<list of change-ids>]` → 这些 change 的 § 9 已处理过，跳过

输出范围声明：

```
✅ A-evolve 扫描范围：
  - 上次同步：<YYYY-MM-DD> · 已处理 <N> 个 change
  - 本次扫：<起始日期> 之后归档的 <M> 个 change
    · archive/2026-04-12-add-cache-layer
    · archive/2026-04-20-fix-auth-flow
    · archive/2026-04-28-add-notifications
    · ...（活跃中的 <active-id> · 是否纳入：用户决定）
  - 当前 CONTEXT.md：最近一次更新 <YYYY-MM-DD>，<N> 行
```

如果范围为空（没有新 change 或全部已处理）→ 直接告诉用户"无新沉淀可同步，CONTEXT.md 已最新"，结束。

### 步骤 2 · 抽取所有 § 9 段

对范围内每个 change：

```
grep_search Query="^## 9\\. 架构沉淀建议" SearchPath="<change/DESIGN.md>"
read_file path="<change/DESIGN.md>" offset=<§ 9 起始行> limit=80
```

把每个 change 的 § 9 段**原样**收集起来（带 source change-id 标签），跳过显式写"本 change 无架构层面沉淀建议"的。

> Token 预算：每个 change 的 § 9 大约 30-80 行 · M 个 change 总计 ~30M-80M 行 · 一般 5-10 个 change 一次同步即可。超过 15 个建议拆两次跑。

### 步骤 3 · 聚合分类

按 § 9 的五类聚合（跨 change 合并）：

#### 3.1 新增可复用抽象（聚合到一张总表）

| 路径 | 能力 | 来自 change | 复用建议 | 跨 change 冲突？ |
|---|---|---|---|---|
| `src/lib/cache.ts` | LRU 缓存 | add-cache-layer | 沿用 | — |
| `src/lib/queue.ts` | 任务队列 | add-bg-jobs | 沿用 | — |
| `src/utils/date-fmt.ts` | 日期格式化 | add-notifications | 沿用 | ⚠️ 与既有 `src/utils/date.ts` 重复 → 让用户决策 |

**冲突检测必跑**：每条新抽象都 grep 一下 CONTEXT.md「既有抽象索引」段是否已有同类 → 有则标 ⚠️。

#### 3.2 项目级技术决策（聚合）

| 决策 | 取值 | 来自 change | 影响范围 | 与现 CONTEXT 冲突？ |
|---|---|---|---|---|
| 缓存层选型 | Redis | add-cache-layer | 全栈 | — |
| 后台作业框架 | BullMQ | add-bg-jobs | 服务端 | — |

#### 3.3 跨模块契约（聚合）

```
新增 API：
  - GET/POST /api/cache/* （来自 add-cache-layer）
  - POST /api/notifications （来自 add-notifications）

新增事件：
  - events.cache.invalidated （来自 add-cache-layer）

Schema 变更：
  - 表 cache_entries （来自 add-cache-layer）
  - 表 notifications （来自 add-notifications）
```

#### 3.4 依赖变动（聚合）

| 包 | 版本变动 | 来自 change | 是否替换 |
|---|---|---|---|
| `ioredis` | + 5.4.0 | add-cache-layer | 替换 node-redis |
| `bullmq` | + 5.x | add-bg-jobs | 新增 |

#### 3.5 禁动清单变动（聚合）

```
新增禁动：
  - src/lib/cache.ts 不允许绕过直接 import ioredis （来自 add-cache-layer）

解禁：
  - src/legacy/cache-old.ts 标记 deprecated，可拆 （来自 add-cache-layer）
```

### 步骤 4 · 逐项 review（用户参与）

**这是 A-evolve 的核心环节。不能跳过。**

按上面五类**逐项**问用户：

```
🟢 候选 1/N · 新增抽象 · src/lib/cache.ts
   能力：LRU 缓存封装
   来自：add-cache-layer（archive/2026-04-12）
   建议：append 到 CONTEXT.md「既有抽象索引」段
   AI 检测：与 CONTEXT 现有内容无冲突

   选项：
   ✅ 接受 promote
   ❌ 跳过（理由：例如临时用，不长期保留）
   ✏️ 编辑后 promote（你写新版本）

请回复：1 / 2 / 3
```

每条等用户回复后才进下一条。**严禁**一次性 batch promote 全部。

#### 冲突项必须显式问

冲突标 ⚠️ 的项额外问：

```
⚠️ 冲突候选 · 新抽象 src/utils/date-fmt.ts vs CONTEXT 现有 src/utils/date.ts
   两者能力重叠（都是日期格式化）。

   选项：
   ✅ 替换 — 用 date-fmt.ts 替换 date.ts，CONTEXT 索引指向新的
   ✅ 共存 — CONTEXT 索引同时列两个，注明用途差异
   ❌ 跳过 — 不 promote 新的，date.ts 仍是单一源
   ✏️ 编辑后定夺

请回复：1 / 2 / 3 / 4
```

**步骤 5–7 · 生成 patch → 写入项目级文档（CONTEXT / ARCHITECTURE）→ EVOLVE 报告 + 更新 STATE**：步骤 4 全部逐项 review 完成后 → **整读 `@code-kit/prompts/A-evolve-patch.md`（131 行）按其执行；不读禁止落盘 patch、禁止更新 STATE。**

---

## 输出

- `.specs/CONTEXT.md`（patch 后 · 用户选 1/3 时）
- `.specs/ARCHITECTURE.md`（patch 后 · 用户选 1/4 时且文件存在）
- 备份文件（二者之一或两者都有）
- `.specs/evolve/<YYYY-MM-DD>-EVOLVE.md`（报告 · 必产）
- 更新的仓库根 `STATE.md`（`last_evolve_at` + `last_evolve_promoted`）

## 约束

- **不动业务代码**：本工作流只 patch `.specs/` 内的项目级文档，禁止改 `src/` / `tests/` / `package.json`
- **逐项 review 强制**：批量 promote 是禁的（容易把烂决策固化进 CONTEXT）
- **冲突必显式问**：标了 ⚠️ 的项不允许默默走「接受」分支
- **备份必跑**：写任何项目级文档前必须先 `cp` 备份
- **不删已有内容**：A-evolve 只 append / update，不 delete（如果发现某段过时该删 / 某 ADR 该 deprecated，写在报告里建议用户跑 A-architect，不自动删）
- **不读 § 9 以外的 DESIGN 内容**：避免把 change 级冻结决策错误升级到项目级
- **遇到架构级冲突停下来**：例如某 § 9 要求 deprecate 现有 ADR / 改依赖规则 → 不在本工作流动，提示用户跑 A-architect

## 自检

- [ ] 已读 `STATE.md` 的 `last_evolve_at` 决定扫描范围
- [ ] 范围内每个 change 都尝试读了 § 9（即使是"无建议"也已记录跳过）
- [ ] 五类聚合表完整（抽象 / 决策 / 契约 / 依赖 / 禁动清单）
- [ ] 冲突项已检测并显式标 ⚠️
- [ ] 逐项 review 完成，每条都有用户的 1/2/3 回复
- [ ] 检查了 ARCHITECTURE.md 是否存在（决定是否生成 `A-evolve-patch.md` § 5.2 patch）
- [ ] 最终 patch 已让用户一次性确认（可选 1/2/3/4/5 任一）
- [ ] 写任何文档前都备份了
- [ ] ADR 编号顺接现有 max + 1，未二次使用已有编号
- [ ] EVOLVE 报告已归入 `.specs/evolve/`
- [ ] STATE.md `last_evolve_at` + `last_evolve_promoted` 已更新
- [ ] 遇到架构级冲突未自作主张，已指引用户跑 A-architect

## 触发下一步

- 同步完成 → 提示用户："下次建议 <YYYY-MM-DD> 同步，或新增 ≥ 5 个 change 后再来"
- 用户选了"暂存 patch 我手动合" → 暂停，告知用户 patch 路径，本工作流结束
- 检测到 ARCHITECTURE.md 不存在但 § 9 里有 ADR 级候选项 ≥ 3 条 → **建议跑 `@A-architect.md` 先建立 ARCHITECTURE.md**，再跑 A-evolve 才能正式 patch ADR
- 检测到 ARCHITECTURE.md ADR 冲突 ≥ 5 条（同主题有多个 accepted ADR）→ 建议跑 `@A-architect.md` 重审
- 检测到 CONTEXT.md 已积累 ≥ 200 行 → 建议用户考虑将部分决策迁到 ARCHITECTURE（剩下 CONTEXT 只记 rules 层）
