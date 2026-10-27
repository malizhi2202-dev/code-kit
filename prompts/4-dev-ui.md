> 分片自 4-dev.md · 何时整读我：任务命中任何用户可见 UI（files 含 css/scss/tsx/vue/jsx/html/svelte，或 action 含颜色/字体/卡片/布局/动画/主题等关键词）时，进入实现前整读并按其执行 · 属 code-kit 小文件按需加载体系（RULES.md R1.10）

### 1.6 UI 任务额外检查（仅当任务涉及任何用户可见 UI）

判定标准：任务的 `files` 包含 `.css` / `.scss` / `.tsx` / `.vue` / `.jsx` / `.html` / `.svelte`，或 `action` 含 button / 颜色 / 字体 / 卡片 / 布局 / 动画 / 主题 等关键词。

命中时，进入实现**之前**还必须：

1. 加载 `@.specs/<id>/UI-DESIGN.md`（必须存在；不存在 → 停下来要求先跑 `@code-kit/prompts/2a-ui-design.md`）
2. 加载 `@code-kit/reference/ui-anti-patterns.md`，按当前任务的关键词 grep 相关章节
3. 加载 `@code-kit/reference/frontend-engineer-rules.md`（**第 1 + 第 2 + 第 10 节必读**），其他节按输出类型按需读：
   - 任务是做交互原型 → 补读第 6.1
   - 任务是做幻灯片 / 演示 → 补读第 6.2
   - 任务是做仪表盘 / 数据可视化 → 补读第 6.3
   - 任务有动画 / 交互动效 → 补读第 4 节 + 第 6.4
   - 任务涉及多变体 / 实时调参 → 补读第 5 节（Tweaks 面板）
4. 对每条命中的"强制禁忌"，在执行计划里**显式声明**：
   - 「已知 X 是禁忌（来自 anti-patterns / frontend-rules），本任务不涉及」 或
   - 「已知 X 是禁忌，本任务采用 Y 替代」
5. **Token 来源单一**：颜色 / 字体 / 间距 / 圆角 / 动效必须从 UI-DESIGN.md frontmatter 派生的 CSS variables / theme 文件中取。**禁止**在组件代码里硬编码颜色或字号（frontend-rules 第 2.1 节）
6. **React 三条硬规则马上写入计划**（仅 React 任务）：禁 `const styles` / 跨文件用 `Object.assign(window, ...)` / 禁 `scrollIntoView`（frontend-rules 第 1.1–1.3）
7. 实现完成后再扫一遍 anti-patterns + frontend-rules 第 10 节交付清单（self-review），写入 SUMMARY.md
