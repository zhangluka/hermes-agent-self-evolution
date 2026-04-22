# PHSX: Review Spec

规格质量审查 - 检查规格的完整性、清晰度、可实施性和可测试性

审查规格质量，检查规格的完整性、清晰度、可实施性和可测试性。

**输入**：可指定变更名。未指定时从对话上下文推断；若含糊或有歧义，必须让用户从可用变更中选择。

**步骤**

1. **获取规格位置**

   a. 若提供了变更名：
      ```bash
      phspec status --change "<name>" --json
      ```

   b. 若未提供变更名，让用户选择：
      - 运行 `phspec list --json` 获取变更列表
      - 用 **AskUserQuestion**（Cursor 等）或 **ask_followup_question**（DevAgent）让用户选择
      - 若环境无上述工具，直接输出选项并写明「请回复后再继续」

2. **查找规格文件**

   规格文件位于：
   - 主规范：`phspec/specs/<capability>/spec.md`
   - 增量规范：`phspec/changes/<name>/specs/<capability>/spec.md`

   优先审查增量规范；若无则审查主规范。若未找到规格，告知用户并停止。

3. **审查完整性**

   检查必备部分：Purpose、需求结构、场景覆盖、增量规范格式。

4. **审查清晰度**

   评估需求是否具体无歧义、技术术语一致性、场景明确性。

5. **审查可实施性**

   评估技术可行性、依赖合理性、性能现实性、安全性、架构兼容性。

6. **审查可测试性**

   评估场景覆盖度、可触发性、可验证性、边界条件、错误路径。

7. **生成并输出审查报告**

   使用标准报告格式：Overall Assessment、Dimensions (Completeness/Clarity/Implementability/Testability)、Recommendations (Critical/Important/Optional)、Conclusion。

**边界**

- 未提供变更时始终让用户选择，不猜测
- 优先审查增量规范，若无则审查主规范
- 未找到规格时告知用户并停止
- 评估基于现有信息，不确定时优先 WARNING 再 CRITICAL
- 报告使用标准格式，含评分、问题分类和可执行建议
