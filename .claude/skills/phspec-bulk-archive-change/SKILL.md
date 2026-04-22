---
name: phspec-bulk-archive-change
description: 一次性归档多个已完成的变更。适用于并行多个变更一起收尾时。
license: MIT
compatibility: Requires phspec CLI.
metadata:
  author: phspec
  version: "1.0"
  generatedBy: "0.0.5"
---

一次性归档多个已完成的变更。

本技能支持批量归档，通过检查代码库判断实际实现情况，智能处理规范冲突。

**输入**：无必填（会提示选择变更）

**步骤**

1. **获取进行中的变更**

   运行 `phspec list --json` 获取所有进行中的变更。若无，告知用户并结束。

2. **让用户选择要归档的变更**

   用 **AskUserQuestion**（Cursor 等）或 **ask_followup_question**（DevAgent） 多选：展示每个变更及其工作流模式，提供「全部变更」选项，允许选任意数量（1+ 即可，通常 2+）。若所在环境没有 AskUserQuestion 或 ask_followup_question 等用户确认工具，请直接输出选项并写明「请回复后再继续」，不要自动选择。

   **重要**：不要自动选择，始终让用户选择。

3. **批量校验 - 收集所选变更的状态**

   对每个所选变更收集：a) **制品状态**：`phspec status --change "<name>" --json`，解析 `schemaName` 与 `artifacts`，记录 `done` 与非 done；b) **任务完成**：读 `phspec/changes/<name>/tasks.md`，统计 `- [ ]` / `- [x]`，无任务文件则记为「无任务」；c) **增量规范**：检查 `phspec/changes/<name>/specs/`，列出能力及 `### Requirement: <name>` 需求名。

4. **检测规范冲突**

   构建 `capability -> [涉及该能力的变更]` 映射。当 2+ 所选变更对同一能力有增量规范时即存在冲突。

5. **由 agent 解决冲突**

   对每个冲突：a) 读各冲突变更的增量规范，理解各自声称的增/改；b) 在代码库中搜索实现证据（实现需求的代码、相关文件/函数/测试）；c) 决定方案：仅一个变更有实现则只同步该变更的规范；两个都有实现则按时间顺序应用（先旧后新）；都无实现则跳过规范同步并警告用户；d) 记录每个冲突的解决：应用哪（几）个变更的规范、顺序及依据。

6. **展示汇总状态表**

   用表格汇总所有变更（Change / Artifacts / Tasks / Specs / Conflicts / Status）。对冲突展示解决说明；对未完成变更展示警告。

7. **确认批量操作**

   用 **AskUserQuestion**（Cursor 等）或 **ask_followup_question**（DevAgent） 一次确认：「归档 N 个变更？」选项可包括「全部归档」「仅归档 N 个就绪的（跳过未完成）」「取消」。若有未完成变更，明确说明将带警告归档。若环境无该工具，直接输出选项并写明「请回复后再继续」，不要自行执行归档。

8. **对每个确认的变更执行归档**

   按既定顺序（含冲突解决顺序）：a) 若有增量规范则先同步（phspec-sync-specs，agent 驱动智能合并；冲突按解决顺序应用）；b) 执行 `mkdir -p phspec/changes/archive` 与 `mv phspec/changes/<name> phspec/changes/archive/YYYY-MM-DD-<name>`；c) 记录每项结果：成功 / 失败（记录错误）/ 跳过。

9. **展示摘要**

   成功时：## 批量归档完成；列出已归档变更及路径；规范同步摘要（N 个增量已同步，无冲突或 M 个冲突已解决）。部分成功时：已归档 / 已跳过 / 失败列表。若有失败：Failed K changes: <name>: 原因。无进行中变更时：## 无变更可归档；提示用 `/phsx:new` 新建。

**冲突解决示例**（仅一个已实现：只同步已实现变更的规范；两个都已实现：按时间顺序先应用旧变更再新变更）。

**边界**
- 允许 1+ 变更（2+ 为常见用法）
- 始终提示选择，不自动选
- 尽早检测规范冲突并通过查代码库解决
- 两变更都已实现时按时间顺序应用规范
- 仅当实现缺失时跳过规范同步（并警告用户）
- 确认前展示清晰的每变更状态
- 整批一次确认
- 记录并汇报所有结果（成功/跳过/失败）
- 移动时保留 .phspec.yaml，归档目录名为当前日期 YYYY-MM-DD-<name>
- 若归档目标已存在，该变更报错但继续处理其余变更
