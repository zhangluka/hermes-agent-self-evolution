---
name: phspec-sync-specs
description: 将变更中的增量规范同步到主规范。适用于用户想更新主规范但暂不归档变更时。
license: MIT
compatibility: Requires phspec CLI.
metadata:
  author: phspec
  version: "1.0"
  generatedBy: "0.0.5"
---

将变更中的增量规范同步到主规范。

此为**由 agent 执行**的操作：你读取增量规范并直接编辑主规范以应用变更，从而可智能合并（例如只加场景而不整条复制需求）。

**输入**：可指定变更名。未指定时从对话推断；含糊或有歧义时必须让用户从可用变更中选择。

**步骤**

1. **若未提供变更名，让用户选择**

   运行 `phspec list --json` 获取变更列表，用 **AskUserQuestion**（Cursor 等）或 **ask_followup_question**（DevAgent） 让用户选择。只展示在 `specs/` 下有增量规范的变更。若所在环境没有 AskUserQuestion 或 ask_followup_question 等用户确认工具，请直接输出变更选项并写明「请回复后再继续」，不要猜测或自动选择。

   **重要**：不要猜测或自动选择，始终让用户选择。

2. **定位增量规范**

   在 `phspec/changes/<name>/specs/*/spec.md` 查找增量规范文件。

   每个增量规范文件包含如：`## ADDED Requirements`（新增需求）、`## MODIFIED Requirements`（修改现有需求）、`## REMOVED Requirements`（删除需求）、`## RENAMED Requirements`（重命名，FROM:/TO: 格式）。

   若未找到增量规范，告知用户并停止。

3. **对每个增量规范，将变更应用到主规范**

   对 `phspec/changes/<name>/specs/<capability>/spec.md` 存在的每个能力：

   a. **读增量规范**，理解要做的变更  
   b. **读主规范** `phspec/specs/<capability>/spec.md`（可能尚不存在）  
   c. **按意图合并**：ADDED 若主规范无则添加、有则按 MODIFIED 更新；MODIFIED 在主规范中找到对应需求后增/改场景或描述，保留增量未提及的内容；REMOVED 从主规范删除整条需求；RENAMED 将 FROM 改为 TO  
   d. **若能力尚无主规范**：创建 `phspec/specs/<capability>/spec.md`，含 Purpose（可简写为 TBD）与 ADDED 需求

4. **展示摘要**：说明更新了哪些能力、做了哪些增/改/删/重命名。

**增量规范格式参考**

```markdown
## ADDED Requirements

### Requirement: New Feature
The system SHALL do something new.

#### Scenario: Basic case
- **WHEN** user does X
- **THEN** system does Y

## MODIFIED Requirements

### Requirement: Existing Feature
#### Scenario: New scenario to add
- **WHEN** user does A
- **THEN** system does B

## REMOVED Requirements

### Requirement: Deprecated Feature

## RENAMED Requirements

- FROM: `### Requirement: Old Name`
- TO: `### Requirement: New Name`
```

**原则：智能合并**——可做部分更新（例如在 MODIFIED 下只加场景、不复制整条）；增量表示*意图*而非整份替换；合理判断合并。

**成功时输出**：摘要「## Specs 已同步：<change-name>」、更新了哪些能力与需求（Added/Modified/Removed/Renamed）、说明主规范已更新、变更仍进行中，实施完成后再归档。

**边界**：先读增量与主规范再改；保留增量未提及的既有内容；不清处先澄清；边改边说明；操作应幂等（执行两次结果一致）。
