---
name: phspec-continue-change
description: 继续当前变更：创建下一个制品。适用于用户想推进变更、创建下一份制品或继续工作流时。
license: MIT
compatibility: Requires phspec CLI.
metadata:
  author: phspec
  version: "1.0"
  generatedBy: "0.0.5"
---

继续当前变更：创建下一个制品。

**执行约定（DevAgent / Cline 等）**：本技能每次调用**只创建一个制品**。创建完该制品后必须先调用用户确认工具（DevAgent：`ask_followup_question`；Cursor 等：`AskUserQuestion`）询问是否继续或要修改，**调用后立即结束本次执行**。不得在本轮中连续创建第二个制品。

**输入**：可指定变更名。未指定时从对话上下文推断；若含糊或有歧义，必须让用户从可用变更中选择。

**步骤**

1. **若未提供变更名，让用户选择**

   运行 `phspec list --json` 获取按最近修改排序的变更列表，再用 **AskUserQuestion**（Cursor 等）或 **ask_followup_question**（DevAgent） 让用户选择要继续的变更。若所在环境没有 AskUserQuestion 或 ask_followup_question 等用户确认工具，请直接输出 3～4 个变更选项并写明「请回复后再继续」，不要猜测或自动选变更。

   将最近修改的 3～4 个变更作为选项展示，包含：
   - 变更名
   - 工作流模式（有 `schema` 字段则用其值，否则为 "spec-driven")
   - 状态（如 "0/5 tasks"、"complete"、"no tasks"）
   - 最近修改时间（来自 `lastModified` 字段）

   将最近修改的变更标为「推荐」，因为用户多半会选它。

   **重要**：不要猜测或自动选变更，始终让用户选择。

2. **查看当前状态**
   ```bash
   phspec status --change "<name>" --json
   ```
   解析 JSON 了解当前状态。响应包含：
   - `schemaName`：所用工作流模式（如 "spec-driven"）
   - `artifacts`：制品数组及状态（"done"、"ready"、"blocked"）
   - `isComplete`：是否全部制品已完成

3. **按状态行动**：

   ---

   **若全部制品已完成（`isComplete: true`）**：
   - 祝贺用户
   - 展示最终状态（含所用模式）
   - 建议："全部制品已就绪！可以用 `/phsx:apply` 实施或归档。"
   - 停止

   ---

   **若有制品可创建**（状态中存在 `status: "ready"` 的制品）：
   - 从状态输出中选第一个 `status: "ready"` 的制品
   - 获取其指令：
     ```bash
     phspec instructions <artifact-id> --change "<name>" --json
     ```
   - 解析 JSON。关键字段：
     - `context`：项目背景（给你的约束，不要写入产出）
     - `rules`：制品级规则（给你的约束，不要写入产出）
     - `template`：产出文件结构
     - `instruction`：模式相关指引
     - `outputPath`：制品写入路径
     - `dependencies`：需先阅读的已完成制品
   - **创建制品文件**：
     - 先读依赖制品以获取上下文
     - 按 `template` 填好各节
     - 写作时遵守 `context` 与 `rules`，但不要原样抄进文件
     - 写入指令中的 outputPath
   - 说明创建了什么、接下来可做哪些。然后**必须结束本次执行**：
     - **在 DevAgent 中**：必须先调用 **`ask_followup_question`**，例如：「已创建 <artifact-id>。要修改刚写的内容，还是继续创建下一个制品？回复继续或说明要改的地方。」**调用后立即结束**，不得在本轮中创建下一个制品。
     - **在 Cursor 等环境中**：使用 **AskUserQuestion** 等价操作后结束。
     - **若环境无上述工具**：输出上述问题文字并写明「请回复后再继续」，然后结束。

   ---

   **若没有可创建制品（全部 blocked）**：
   - 在合法模式下不应出现
   - 展示状态并建议检查问题

4. **创建制品后展示进度**
   ```bash
   phspec status --change "<name>"
   ```

**输出**

每次调用后展示：
- 创建了哪个制品
- 所用工作流模式
- 当前进度（N/M 已完成）
- 当前可创建的制品
- 提示："要继续吗？说继续或告诉我下一步即可。"

**制品创建指引**

制品类型与用途由模式决定。以指令输出中的 `instruction` 为准。

常见模式（spec-driven）：proposal → specs → design → tasks
- **proposal.md**：若不清楚可先问用户，填写 Why、What Changes、Capabilities、Impact。Capabilities 很关键，每项能力对应一个 spec 文件。若环境无 AskUserQuestion 或 ask_followup_question，直接输出问题并写明「请回复后再继续」，不要自行假设后继续。
- **specs/<capability>/spec.md**：按提案 Capabilities 每项建一个规范（用能力名，不是变更名）。
- **design.md**：记录技术决策、架构与实现思路。
- **tasks.md**：把实现拆成可勾选任务。

其他模式以 CLI 输出的 `instruction` 为准。

**边界**
- 每次调用**仅**创建一个制品；创建完一个制品后必须停止，等用户说「继续」或再次调用本技能后再创建下一个；不得在本轮中连续创建多个制品。
- 创建新制品前先读依赖制品
- 不跳过、不乱序
- 上下文不清时先问用户
- 写入后确认文件存在再标记进度
- 按模式的制品顺序来，不假设具体名称
- **重要**：`context` 与 `rules` 是给你的约束，不是文件内容；不要将 `<context>`、`<rules>`、`<project_context>` 抄进制品，它们只指导写作，不得出现在产出中
