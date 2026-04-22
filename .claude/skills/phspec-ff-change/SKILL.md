---
name: phspec-ff-change
description: 快进式创建 PhSpec 制品。适用于用户想一次性生成实施所需全部制品时。
license: MIT
compatibility: Requires phspec CLI.
metadata:
  author: phspec
  version: "1.0"
  generatedBy: "0.0.5"
---

快进式创建制品——一次性生成实施所需全部制品。

**输入**：用户请求应包含变更名（kebab-case）或要做的内容描述。

**步骤**

1. **若未提供明确输入，先问用户要做什么**

   使用 **AskUserQuestion**（Cursor 等）或 **ask_followup_question**（DevAgent）（开放问题）询问：
   > "你想做哪个变更？描述你想做或要修的内容。"

   若所在环境没有 AskUserQuestion 或 ask_followup_question 等用户确认工具，请直接输出该问题并写明「请回复后再继续」，不要自行假设名称后继续。

   从其描述推导 kebab-case 名称（如 "add user authentication" → `add-user-auth`）。

   **重要**：未弄清用户要做什么前不要继续。

2. **创建变更目录**
   ```bash
   phspec new change "<name>"
   ```
   会在 `phspec/changes/<name>/` 下创建脚手架。

3. **获取制品构建顺序**
   ```bash
   phspec status --change "<name>" --json
   ```
   解析 JSON 得到：
   - `applyRequires`：实施前需要的制品 ID 列表（如 `["tasks"]`）
   - `artifacts`：所有制品及其状态与依赖

4. **按顺序创建制品直至可实施**

   用 **TodoWrite 工具** 跟踪进度。

   按依赖顺序遍历（无未满足依赖的制品先做）：

   a. **对每个 `ready`（依赖已满足）的制品**：
      - 获取指令：
        ```bash
        phspec instructions <artifact-id> --change "<name>" --json
        ```
      - 指令 JSON 含：`context`（项目背景，勿写入产出）、`rules`（制品规则，勿写入产出）、`template`（产出结构）、`instruction`（该类型指引）、`outputPath`、`dependencies`（需先读的制品）
      - 先读依赖制品，再按 `template` 创建文件，遵守 `context` 与 `rules` 但不抄入文件
      - 简要提示："✓ 已创建 <artifact-id>"

   b. **直到 `applyRequires` 中制品全部完成**
      - 每创建一个制品后重跑 `phspec status --change "<name>" --json`
      - 当 `applyRequires` 中每个 ID 在 artifacts 中均为 `status: "done"` 时停止

   c. **若某制品需要用户输入**（上下文不清）：用 **AskUserQuestion**（Cursor 等）或 **ask_followup_question**（DevAgent） 澄清后继续；若环境无该工具，直接输出问题并写明「请回复后再继续」，不要自行假设后继续。

5. **展示最终状态**
   ```bash
   phspec status --change "<name>"
   ```

**输出**：总结变更名与路径、已创建制品列表、"全部制品已就绪，可以开始实施。"、提示 "运行 `/phsx:apply` 或让我实施即可开始任务。"

**制品创建**：按 `phspec instructions` 的 `instruction` 与模式定义；先读依赖再创建；`context` 与 `rules` 仅作约束不写入文件。

**边界**：创建模式 `apply.requires` 所需的全部制品；先读依赖再创建；名称已存在时建议继续该变更；写入后确认文件存在再继续。
