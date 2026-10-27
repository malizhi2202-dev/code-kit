> 分片自 A-evolve.md · 何时整读我：步骤 4 逐项 review 批准项收集完成后，进入生成 patch、写项目级文档、输出 EVOLVE 报告与更新 STATE（原步骤 5–7）时整读并按其执行；未读禁止写 CONTEXT / ARCHITECTURE / STATE · 属 code-kit 小文件按需加载体系（RULES.md R1.10）

### 步骤 5 · 生成 patch（合并入 CONTEXT.md · 可选 ARCHITECTURE.md）

收集所有用户批准的项，生成**两份 patch**（项目有 ARCHITECTURE 则两份都生，否则只生 CONTEXT 那份）。

#### 5.1 patch CONTEXT.md（rules 层 · 必生）

```markdown
## CONTEXT.md patch · YYYY-MM-DD

### 「既有抽象索引」段 append
+ src/lib/cache.ts · LRU 缓存 · 来源 add-cache-layer (2026-04-12)
+ src/lib/queue.ts · 任务队列 · 来源 add-bg-jobs (2026-04-20)

### 「已锁技术决策」段 append
+ 缓存层：Redis（替代原 in-memory）· 来源 add-cache-layer
+ 后台作业框架：BullMQ · 来源 add-bg-jobs

### 「技术栈」段 update
~ ioredis 5.4.0 替换 node-redis（旧版可拆）
+ bullmq 5.x 新增

### 「禁动清单」段 update
+ src/lib/cache.ts 不允许绕过直接 import ioredis
- src/legacy/cache-old.ts（标 deprecated · 待拆）
```

#### 5.2 patch ARCHITECTURE.md（structure 层 · 仅在 ARCHITECTURE.md 存在时生）

```markdown
## ARCHITECTURE.md patch · YYYY-MM-DD

### § 3 ADR 列表 · 新增
+ ADR-008 · 缓存层：Redis
  - 状态：accepted (YYYY-MM-DD)
  - 取舍：in-memory LRU / Redis / Memcached
  - 决定：Redis 7（ioredis 客户端）
  - 理由：多实例部署需共享缓存
  - 代价：多一个运维组件
  - 来源：add-cache-layer
  - 推翻成本：中

+ ADR-009 · 后台作业框架：BullMQ
  - ...

### § 4.1 公共 HTTP API · append
+ /api/cache/*  ← add-cache-layer
+ /api/notifications/*  ← add-notifications

### § 4.2 事件总线 · append
+ events.cache.invalidated  · 来源 add-cache-layer

### § 4.3 数据库 schema · 提及新表
+ cache_entries（详细 schema 见 add-cache-layer migration）
+ notifications（详细 schema 见 add-notifications migration）

### § 8 修订历史 · append
| YYYY-MM-DD | A-evolve | ADR-008/009 新增（来自 add-cache-layer / add-bg-jobs）| A-evolve |
```

**两者分工**：

- 技术栈变动 / 抽象索引 / 禁动清单 → 只进 CONTEXT（这些是 AI 实施层会读的）
- ADR / 跨模块契约 / Schema 总览 / 修订历史 → 只进 ARCHITECTURE（这些是人读 / 2-design 读的）
- 项目级技术决策（如“缓存选 Redis”）**两边都进**：CONTEXT 记一句话供 AI 快查，ARCHITECTURE 记完整 ADR

#### 5.3 给用户最终 review

```
✅ 即将应用的 patch（CONTEXT 共 N 条，ARCHITECTURE 共 M 条）：
[先贴 CONTEXT patch]
[再贴 ARCHITECTURE patch · 如有]

确认应用？
1. ✅ 应用两边
2. ⏸️ 暂存 patch 为 .specs/evolve/<YYYY-MM-DD>-EVOLVE-PATCH.md，我手动合
3. ✅ 只应用 CONTEXT 不应用 ARCHITECTURE
4. ✅ 只应用 ARCHITECTURE 不应用 CONTEXT
5. ❌ 取消，啥都不动
```

### 步骤 6 · 写入项目级文档（用户选 1/3/4）

**安全要求**：

- 写入前先备份（双选动则两份都备）：
  - `cp .specs/CONTEXT.md .specs/CONTEXT.md.bak-<YYYY-MM-DD>`
  - `cp .specs/ARCHITECTURE.md .specs/ARCHITECTURE.md.bak-<YYYY-MM-DD>`（如存在）
- 用 `edit` 工具按段 append / update（**不要整文件 rewrite**，避免破坏未涉及的内容）
- 每段 append 在末尾加注释：`<!-- A-evolve YYYY-MM-DD: from <change-id> -->`
- ADR 编号**顺接现有最大值**（grep `^### ADR-\d+` ARCHITECTURE.md 取 max + 1）

### 步骤 7 · 输出 EVOLVE 报告 + 更新 STATE

#### 7.1 报告写入 `.specs/evolve/<YYYY-MM-DD>-EVOLVE.md`

```markdown
# 架构演进同步 · YYYY-MM-DD

## 扫描范围
- 起始：上次同步 <date> 之后
- 涵盖 change：[列表]

## 候选汇总
- 新抽象：M 条 · 接受 X · 跳过 Y · 编辑 Z
- 技术决策：M 条 · ...
- 跨模块契约：...
- 依赖变动：...
- 禁动清单：...

## 应用 patch
[贴步骤 5 的最终 patch]

## 跳过项 + 理由
- src/utils/date-fmt.ts → 跳过，理由：与既有 date.ts 重复，保留单一源

## 下次建议同步时间
建议 <YYYY-MM-DD>（约 60 天后），或在新增 ≥ 5 个有 § 9 内容的 change 之后
```

#### 7.2 更新 `STATE.md`

```yaml
last_evolve_at: <YYYY-MM-DD>
last_evolve_promoted:
  - add-cache-layer
  - add-bg-jobs
  - add-notifications
  # ... 本次涉及的所有 change-id（无论是否被 promote，只要扫过就记，避免下次重复扫）
```
