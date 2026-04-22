# PHSX: Fast Forward

创建变更并一次性生成实施所需全部制品

快进式创建制品——一次性生成实施所需全部制品。

**输入**：`/phsx:ff` 后为变更名（kebab-case）或用户想做什么的描述。

**步骤**

1. **若未提供明确输入，先问用户要做什么**

   用 **AskUserQuestion**（Cursor 等）或 **ask_followup_question**（DevAgent） 询问并推导 kebab-case 名称。若所在环境没有 AskUserQuestion 或 ask_followup_question 等用户确认工具，请直接输出问题并写明「请回复后再继续」，不要自行假设后继续。**重要**：未弄清用户要做什么前不要继续。

2. **创建变更目录**
   ```bash
   phspec new change "<name>"
   ```
   会在 `phspec/changes/<name>/` 下创建脚手架。

3. **获取制品构建顺序**
   ```bash
   phspec status --change "<name>" --json
   ```
   解析 `applyRequires`（实施前需要的制品 ID）、`artifacts`（所有制品及状态与依赖）。

4. **按顺序创建制品直至可实施**

   用 **TodoWrite 工具** 跟踪进度。按依赖顺序遍历：对每个 `ready` 制品运行 `phspec instructions <artifact-id> --change "<name>" --json`，解析 context/rules/template/instruction/outputPath/dependencies；先读依赖制品，按 template 创建文件，遵守 context 与 rules 但不抄入文件；简要提示「✓ 已创建 <artifact-id>」。每创建一个制品后重跑 status，当 `applyRequires` 中制品均为 `done` 时停止。若某制品需用户输入则用 AskUserQuestion 或 ask_followup_question 澄清后继续；若环境无该工具，直接输出问题并写明「请回复后再继续」，不要自行假设后继续。

5. **展示最终状态**
   ```bash
   phspec status --change "<name>"
   ```

**输出**：总结变更名与路径、已创建制品列表、「全部制品已就绪，可以开始实施。」、提示「运行 `/phsx:apply` 或让我实施即可开始任务。」

**制品创建**：按 `phspec instructions` 的 `instruction` 与模式定义。**边界**：创建模式 `apply.requires` 所需的全部制品；先读依赖再创建；名称已存在时建议继续该变更；写入后确认文件存在再继续。
