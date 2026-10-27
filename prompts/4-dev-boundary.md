> 分片自 4-dev.md · 何时整读我：每个任务 verify 通过、提交前整读并按其执行 diff 边界检查（R6.5 强制，任何生产代码任务都要跑） · 属 code-kit 小文件按需加载体系（RULES.md R1.10）

### 5. 提交前 diff 边界 verify（强制 · 对应 R6.5 / B3 老项目护栏）

> 防"AI 顺手改了别的"。提交前必须验证 diff 范围 ⊆ TASK 的 `write_files`。

#### 5.1 跑 diff 检查

```bash
# 列出实际 diff 涉及的文件
git diff --name-only HEAD
git diff --cached --name-only       # 含 staged
git status --short                  # 含 untracked
```

#### 5.2 比对 TASK 的 write_files

把上面输出与 `TASK.md` 当前 task 的 `<write_files>` 字段比对：

```
✅ TASK 声明的 write_files：
  - src/features/notifications/NotificationCenter.tsx
  - src/features/notifications/useNotifications.ts
  - src/features/notifications/__tests__/*

✅ 实际 diff 涉及：
  - src/features/notifications/NotificationCenter.tsx
  - src/features/notifications/useNotifications.ts
  - src/features/notifications/__tests__/NotificationCenter.test.tsx

→ 0 越界 ✅
```

或者：

```
⚠️ 越界检测：

✅ TASK 声明的 write_files：
  - src/features/notifications/*

❌ 实际 diff 越界文件：
  - src/components/Layout.tsx（DESIGN 0.5.1 「禁动清单」中的文件）
  - src/api/admin/users/route.ts（不在 write_files 范围内）

→ 必须停下来：
  选项 1. 撤销越界改动（git checkout -- <files>）
  选项 2. 更新 TASK 的 write_files（须人工同意，相当于扩范围）
  选项 3. 把越界改动拆成新 task / 新 CHANGE
```

#### 5.3 验证结果写入 SUMMARY「越界检查」段

即使 0 越界也要写：

```
✅ 越界检查（R6.5）：
  - TASK write_files：3 项
  - 实际 diff 涉及：3 项
  - 越界：0
```

**禁止**："顺手修了个 bug" / "看到这里很丑就改了"——必须开新 task 或新 CHANGE。
