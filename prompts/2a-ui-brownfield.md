> 分片自 2a-ui-design.md · 何时整读我：步骤 0 判定为 brownfield（既有项目加 / 改 UI）时，进步骤 2 之前整读并按其执行视觉语汇对齐（R8.7 强制） · 属 code-kit 小文件按需加载体系（RULES.md R1.10）

### 1.5 brownfield 视觉语汇对齐（旧项目加 UI 必走 · 对应老项目护栏 B2 的视觉版）

> 目标：**新增的元素和原有的在视觉上无法区分**。没这一步，AI 很容易把一个正文项目混进典型 AI-slop 风格。

#### 1.5.1 挖出现有视觉语汇（电影院模式：思考出声，让用户能校准观察结果）

优先读代码 ≫ 看截图。按下面顺序提取：

1. **Token 源**： grep `src/**/tokens.{ts,css}` / `theme.ts` / `tailwind.config.{js,ts}` / `:root` / `--color-` / `--font-` 找已有的 design token
2. **实际色彩比例**：打开 3–5 个代表页面，记录主色 / 中性 / 强调色的使用比例（不是看 token 定义，是看用的比例）
3. **交互反馈语言**： grep `hover:` / `:hover` / `focus:` / `transition` 在既有组件里怎么用的（颜色变 / 阴影 / transform / opacity）
4. **动效语言**： grep `cubic-bezier` / `ease-` / `duration-` / `@keyframes` 找缓动曲线 + 时长档位，确认是 CSS 过渡 / CSS 动画 / JS 驱动
5. **结构语言**： elevation 层级有几个（阴影 token 数量），卡片密度稀/密，rounded 统一还是分级，常见布局模式（分栏 / 卡片网格 / 时间线 / 表格）
6. **图形与图标**：用的是哪个图标库（`lucide` / `heroicons` / `phosphor` / 自绘 SVG 集？），插画风格，图片处理手法
7. **文案调性**：工程向 / 营销向 / 中性？按钮文案长什么样（"创建" vs "立即开始免费体验"）

把上面 7 项形成一份「观察报告」**贴给用户校准**：

```
🔍 视觉语汇观察（你看看对不对）：
- 主色：--color-accent oklch(0.65 0.18 230)，实际页面上只在 primary button 和链接用，约占 3%
- 中性：背景 oklch(0.98 0.01 80) 偏米白；文字用 oklch(0.25 0.01 80)；**不用纯黑纯白**
- hover：统一是颜色变深 + transform: translateY(-1px)，duration 150ms，cubic-bezier(0.16, 1, 0.3, 1)
- elevation：只有 2 级（shadow-sm / shadow-lg），无多层纵深
- 卡片密度：稀、内边距 24px，rounded-lg，不用渐变
- 图标：lucide-react，stroke-width 1.5
- 文案：工程向，动词为主（"Configure" 而非 "Set it up"）

你确认我看对了吗？有没有误读的点？
```

#### 1.5.2 用户确认后再进步骤 2

- 用户修正了观察 → 记录修正，重贴一次确认
- 用户说没问题 → 进步骤 2，但**5 维决策的默认势能调成"沿用观察到的"**，不再从空气里选
- 用户明确说"本次想远离原有风格"（比如做 promo 页 / 节日特别版）→ 保留观察作为"约束边界"（字体不别差太远，但配色可以破）

#### 1.5.3 写入 UI-DESIGN.md

在 `UI-DESIGN.md` 的 `## 0. 视觉语汇对齐`（新增段，仅 brownfield 填）贴观察报告 + 用户校准结论。
