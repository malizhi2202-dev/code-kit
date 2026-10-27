> 分片自 4-dev.md · 何时整读我：本任务 diff 命中删 ≥5 行 / 改公共导出 / 改公共 API / 删文件或重命名导出（R4.6）任一条件时，动手前整读并按其执行 · 属 code-kit 小文件按需加载体系（RULES.md R1.10）

### 1.8 破坏性变更高门槛（强制 · 对应 R4.6 / B4 老项目护栏）

> 删错代码 / 改坏公共接口是**老项目最高频真事故**。本段强制堵住。

**判定标准**：本任务 diff 命中下面**任一**条件 → 必须走本协议：

1. **删除既有代码** ≥ 5 行（不算空行 / 注释）
2. **改公共导出**：导出函数 / 类 / 接口的签名变更（参数 / 返回值 / 类型）
3. **改公共 API**：HTTP / GraphQL / gRPC 路由或 schema 变更
4. **删除文件** 或 重命名导出符号

#### 1.8.1 grep 引用图（必须）

对每个被删 / 改签名的符号，执行 grep：

```bash
# 例：删除 formatLegacyDate 函数
grep -rn "formatLegacyDate" src/ tests/ scripts/ docs/
# 例：改 fetchUser 的签名
grep -rn "fetchUser\b" src/ tests/
# 例：删除文件 src/utils/old-helpers.ts
grep -rn "from.*old-helpers\|import.*old-helpers" src/ tests/
```

**贴出完整结果**（哪怕 0 命中也要贴 0 命中的 grep，证明你查了）。

#### 1.8.2 列出影响清单

把 grep 结果整理成清单，**含间接影响**：

```
被删/改的符号：formatLegacyDate (src/utils/legacy-date.ts:12)
直接调用：
  - src/components/InvoiceTable.tsx:34
  - src/api/reports/route.ts:67
  - tests/legacy-date.test.ts:8
间接影响：
  - InvoiceTable 是 src/pages/invoices.tsx 的子组件 → invoice 页面会受影响
  - reports/route 被 cron job daily-report 触发 → 可能影响定时任务
```

⚠️ **特别注意**：

- AI 看到的 grep 结果 ≠ 真实引用图：动态 import / 反射 / 字符串拼接的 require / 配置驱动的 hook 名都可能漏掉
- 对**公共 API**还要 grep **客户端代码**（mobile app / 第三方集成）
- 对**导出符号**还要 grep **文档 / README / Storybook**

#### 1.8.3 反问用户（停下来）

把 1.8.1 + 1.8.2 的结果**贴出来反问用户**：

```
🔴 检测到破坏性变更：删除 formatLegacyDate（5 行 + 8 处调用点）

调用点清单：
  - src/components/InvoiceTable.tsx:34（调用 1 处）
  - src/api/reports/route.ts:67（调用 1 处）
  - tests/legacy-date.test.ts:8（测试用）

请选择处理方式：
  1. 直接删除 + 同步改 8 处调用点（我会一并改）
  2. 留兼容期：保留 formatLegacyDate 但加 @deprecated 注解 + 6 个月后清理（建议）
  3. 写 codemod：生成 ts-morph / jscodeshift 脚本批量替换（适用调用点 > 20）
  4. 不删了，找别的办法实现需求

无人工确认前，我不动手。
```

#### 1.8.4 回归测试覆盖

无论选哪个方案，必须确保：

- 删除 / 改动的旧路径**有测试覆盖**（不能默默 break）
- 改公共 API 的新旧版本必须**同时有测试**（兼容期内两套都跑）

#### 1.8.5 写入 SUMMARY「破坏性变更」段

把 1.8.1~1.8.4 的全部内容写入 `<task-id>-SUMMARY.md` 的「破坏性变更」段（SUMMARY 模板里有）。

#### 1.8.6 何时**不必**走本协议

- 重构内部实现（导出符号不变）
- 加新参数但保持向后兼容（带默认值的可选参数）
- 删 < 5 行实现细节（无外部引用）
