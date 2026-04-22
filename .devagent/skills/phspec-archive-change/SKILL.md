---
name: phspec-archive-change
description: 归档已完成的变更。适用于实施完成后要收尾并归档变更时。
license: MIT
compatibility: Requires phspec CLI.
metadata:
  author: phspec
  version: "1.0"
  generatedBy: "0.0.5"
---

在实验性工作流中归档已完成的变更。

**输入**：可指定变更名。未指定时从对话推断；含糊或有歧义时必须让用户从可用变更中选择。

**步骤**

1. **若未提供变更名，让用户选择**

   运行 `phspec list --json` 获取变更列表，用 **AskUserQuestion**（Cursor 等）或 **ask_followup_question**（DevAgent） 让用户选择。只展示进行中的变更（未归档的），若有则展示每个变更所用工作流模式。若所在环境没有 AskUserQuestion 或 ask_followup_question 等用户确认工具，请直接输出变更选项并写明「请回复后再继续」，不要猜测或自动选择。

   **重要**：不要猜测或自动选择，始终让用户选择。

2. **检查制品完成状态**

   运行 `phspec status --change "<name>" --json` 查看制品完成情况。解析 JSON：`schemaName`、`artifacts` 及各状态（`done` 或其他）。若有制品未 `done`：列出未完成制品并警告，用 **AskUserQuestion**（Cursor 等）或 **ask_followup_question**（DevAgent） 确认是否继续，用户确认后继续。若环境无该工具，直接输出警告与「是否继续？」并写明「请回复后再继续」，不要自行假设。

3. **检查任务完成状态**

   阅读任务文件（通常为 `tasks.md`），统计 `- [ ]`（未完成）与 `- [x]`（已完成）。若有未完成任务：展示未完成数量并警告，用 **AskUserQuestion**（Cursor 等）或 **ask_followup_question**（DevAgent） 确认是否继续，用户确认后继续。若环境无该工具，直接输出问题并写明「请回复后再继续」。若无任务文件：不提示任务相关警告，继续。

4. **评估增量规范同步状态**

   检查 `phspec/changes/<name>/specs/` 是否有增量规范。若无则不必提示同步。若有：将各增量规范与主规范 `phspec/specs/<capability>/spec.md` 对比，说明会应用哪些变更（增/改/删/重命名），在提示前展示合并摘要。选项：若需同步则「立即同步（推荐）」「不同步直接归档」；若已同步则「立即归档」「仍同步一次」「取消」。若用户选同步，用 Task 工具（subagent_type: "general-purpose", prompt: "用 Skill 工具调用 phspec-sync-specs 处理变更 '<name>'。增量分析：<上述摘要>"）。若环境无 AskUserQuestion 或 ask_followup_question，直接输出选项并写明「请回复后再继续」，不要自行选择并执行。无论是否同步，最终执行归档。

5. **执行归档**

   若不存在则创建 `phspec/changes/archive`。目标名用当前日期：`YYYY-MM-DD-<change-name>`。若目标已存在：报错并建议重命名已有归档或换日期。否则执行 `mv phspec/changes/<name> phspec/changes/archive/YYYY-MM-DD-<name>`。

6. **展示摘要**

   包含：变更名、所用工作流模式、归档路径、是否已同步规范（若适用）、任何警告（未完成制品/任务）。

**成功时输出**

```
## 归档完成

**变更：** <change-name>
**工作流模式：** <schema-name>
**归档至：** phspec/changes/archive/YYYY-MM-DD-<name>/
**规范：** ✓ 已同步到主规范（或「无增量规范」或「已跳过同步」）

全部制品已完成。全部任务已完成。
```

**边界**
- 未提供变更时始终让用户选择
- 用 phspec status --json 检查完成度
- 有警告时仅提示并确认，不阻止归档
- 移动目录时保留 .phspec.yaml
- 若需同步则用 phspec-sync-specs（agent 驱动）
- 有增量规范时先做同步评估并展示合并摘要再提示
