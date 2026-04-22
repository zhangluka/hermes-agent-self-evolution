# PHSX: New

新建变更（PHSX 制品工作流）

使用实验性制品驱动工作流新建变更。

**执行约定（DevAgent / Cline 等 workflow）**：本 workflow 在步骤 6 **必须结束**。步骤 6 中你必须**先调用**用户确认工具（DevAgent：`ask_followup_question`；Cursor 等：`AskUserQuestion`）向用户提问，**调用后立即结束本次执行**，不得在本轮中创建 proposal.md、design.md、specs、tasks.md 等任何制品。用户回复后再由其主动运行 `/phsx:continue` 或说「继续」来创建制品。

**输入**：`/phsx:new` 后的参数为变更名（kebab-case），或用户想做什么的描述。

**步骤**

1. **若未提供输入，先问用户要做什么**

   使用 **AskUserQuestion**（Cursor 等）或 **ask_followup_question**（DevAgent）（开放问题，无预设选项）询问：
   > "你想做哪个变更？描述你想做或要修的内容。"

   从其描述推导出 kebab-case 名称（如 "add user authentication" → `add-user-auth`）。

   **重要**：未弄清用户要做什么前不要继续。

2. **确定工作流模式**

   除非用户明确要求其他工作流，否则使用默认模式（不传 `--schema`）。

   **仅在用户提到以下情况时使用其他模式：**
   - 指定了某模式名 → 使用 `--schema <name>`
   - "show workflows" 或 "what workflows" → 运行 `phspec schemas --json` 让其选择

   **否则**：不传 `--schema`，使用默认。

3. **创建变更目录**
   ```bash
   phspec new change "<name>"
   ```
   仅当用户指定了工作流时才加 `--schema <name>`。
   会在 `phspec/changes/<name>/` 下按所选模式创建脚手架。

4. **查看制品状态**
   ```bash
   phspec status --change "<name>"
   ```
   可看到哪些制品待创建、哪些已就绪（依赖已满足）。

5. **获取第一个制品的指令**
   第一个制品由模式决定（如 spec-driven 下为 `proposal`）。
   在状态输出中找到第一个状态为 "ready" 的制品。
   ```bash
   phspec instructions <first-artifact-id> --change "<name>"
   ```
   会输出创建该制品所需的模板与上下文。

6. **在此暂停，等待用户指示（必须结束）**

   - **在 DevAgent 中**：必须先调用 **`ask_followup_question`** 工具，向用户提问，例如：「变更 <name> 已创建，第一个待建制品是 proposal。要创建第一个制品了吗？直接说说这个变更要做什么，我来起草；或回复继续让我创建。」**调用后立即结束本次 workflow 执行**，不得再执行任何写文件操作。
   - **在 Cursor 等环境中**：使用 **AskUserQuestion** 等价操作，调用后结束本次执行。
   - **若环境无上述工具**：直接输出上述问题文字并写明「请回复后再继续」，然后结束，不要创建任何制品文件。
   - **禁止**：在本轮中创建或写入 proposal.md、design.md、specs/*、tasks.md；禁止在未调用用户确认工具（或输出问题并结束）的情况下继续执行后续步骤。

**输出**

完成上述步骤后总结：
- 变更名与路径
- 所用工作流模式及其制品顺序
- 当前状态（0/N 个制品已完成）
- 第一个制品的模板
- 通过用户确认工具或文字提示询问："要创建第一个制品了吗？直接说说这个变更要做什么，我来起草；或让我继续。"

**边界**
- 本命令职责仅到「展示第一个制品模板并询问用户」；**不得**在本命令中创建任何制品（proposal、specs、design、tasks 等）；步骤 6 必须通过工具或文字询问后**结束**，由用户下次运行 `/phsx:continue` 再创建制品。
- 先不要创建任何制品，只展示指令；不要越过「展示第一个制品模板」这一步。
- 若名称无效（非 kebab-case），请用户给出合法名称
- 若该名称的变更已存在，建议改为继续该变更
- 使用非默认工作流时传入 --schema
