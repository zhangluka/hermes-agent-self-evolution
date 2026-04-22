# PHSX: Verify

归档前校验实现是否与变更制品一致

校验实现是否与变更制品（规范、任务、设计）一致。

**输入**：可指定变更名。未指定时从对话推断；含糊或有歧义时必须让用户从可用变更中选择。

**步骤**

1. **若未提供变更名，让用户选择**：运行 `phspec list --json`，用 **AskUserQuestion**（Cursor 等）或 **ask_followup_question**（DevAgent） 让用户选择。展示有实施任务（存在 tasks 制品）的变更、各变更所用工作流模式，未完成任务标为「进行中」。若环境无该工具，直接输出变更选项并写明「请回复后再继续」，不要猜测或自动选择。**重要**：不要猜测或自动选择。

2. **查看状态以了解工作流**：`phspec status --change "<name>" --json`，解析 `schemaName` 及该变更有哪些制品。

3. **获取变更目录并加载制品**：`phspec instructions apply --change "<name>" --json`，从返回的 `contextFiles` 中读取所有可用制品。

4. **初始化校验报告结构**：按四个维度建报告——**完整性**（任务与规范覆盖）、**正确性**（需求实现与场景覆盖）、**一致性**（设计遵循与模式一致）、**代码卫生**（代码质量与规范合规性）。每个维度可有 CRITICAL、WARNING、SUGGESTION 级别问题。

5. **校验完整性**：若有 tasks.md，解析 `- [ ]` / `- [x]`，未完成任务记 CRITICAL。若有 `phspec/changes/<name>/specs/` 增量规范，提取 `### Requirement:` 需求，逐条在代码库中搜索评估是否已实现；明显未实现则记 CRITICAL。

6. **校验正确性**：对每条需求搜索实现证据，评估是否与需求意图一致；对 `#### Scenario:` 场景检查代码与测试覆盖；检查规范关键字（SHALL、MUST、REQUIRED、WILL）的实现。偏离或未覆盖记 WARNING。

7. **校验一致性**：若有 design.md，提取关键决策并核对实现是否遵循；检查新代码与项目模式是否一致。矛盾记 WARNING，明显偏离记。

8. **校验代码卫生**：检查重复代码、错误处理、注释质量、命名规范。重复代码记 SUGGESTION，缺少错误处理记 WARNING，命名偏离记 SUGGESTION。

9. **生成校验报告**：摘要表（## 校验报告：<change-name>；Completeness/Correctness/Coherence/CodeHygiene 状态）；按 CRITICAL/WARNING/SUGGESTION 列问题，每条带可执行建议及文件/行引用；结论（有 CRITICAL 则「归档前请修复」；仅 WARNING 则「可归档并改进」；全部通过则「可归档」）。

**校验启发**：完整性关注客观清单；正确性用关键词与合理推断；一致性关注明显不一致；代码卫生关注质量改进。不确定时优先 SUGGESTION 再 WARNING 再 CRITICAL。**重要**：本命令为建议式校验，不强制阻塞归档。**降级**：仅 tasks.md 时只校验任务；有 tasks+specs 时校验完整性与正确性；全量制品时校验四维。**输出格式**：清晰 Markdown、摘要表、按优先级分组、`file.ts:123` 引用、具体建议。
