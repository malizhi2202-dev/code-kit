# NOTICE · 来源与许可

本仓库是方法论文档的整合与改写，不是任一上游仓库的完整副本。

## 1. code-kit 本体

- 版权与许可：见根目录 `LICENSE`（MIT License）。
- `GO.md` / `RULES.md` / `METHODOLOGY.md` / `SYSTEM.md` / `prompts/` / `templates/` / `reference/` 中的原有内容属 code-kit 本体。

## 2. multica starterkit skills（新增整合）

- 来源：`multica` 仓库 `server/internal/service/starterkit/fixtures/skills/`（30 个 SKILL.md 及其 references；本仓库收录 29 个）。
- 版权：© 2025–2026 Multica, Inc.（以源仓库 LICENSE / NOTICE 为准）。
- 许可：**Multica License** = Apache License 2.0（Part II）+ Part I 附加条件。
  - 附加条件主要限制：把 Multica 源码作为托管服务提供给第三方；移除 Multica 品牌/版权信息。
  - **组织内部使用**与**公开源码分发**不受托管服务限制，但分发须保留许可与归属信息。
  - 若对外分发本仓库中 multica 衍生的部分（`reference/engineering.md`、`quality.md`、`planning.md`、`reference/automation.md`、`reference/coverage.md` 及 `RULES.md` R17/R18 等），须**随附 Multica License 全文（Part I + Part II）**，即源仓库 `LICENSE` 文件。
- 本仓库只吸收其流程与方法论（已改写为中文并重组），不包含 multica 的产品代码、UI 或运行时。

## 3. ponytail（新增整合）

- 来源：https://github.com/DietrichGebert/ponytail （v5）。
- 许可：**MIT**，Copyright (c) 2026 DietrichGebert。再分发须保留本版权与许可声明。
- `reference/ponytail.md` 为其规则、review/audit/debt 流程的中文改写与精简；原项目还包含基准、hooks、band 等运行时组件，**未**包含在本仓库中。
- ponytail 是独立项目，与本仓库无隶属或背书关系。

## 4. 使用建议

- 本仓库面向**本地 / 组织内部**的 AI agent 使用。
- 若要把 multica 衍生部分对外分发或作为托管服务的一部分，请先核对 Multica License Part I 的条件（必要时取得商业许可），并随附完整 License。
- ponytail 部分再分发时保留上述 MIT 版权声明。
