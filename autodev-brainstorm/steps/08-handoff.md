# 步骤 8：交接

**如果从流水线调用：**
- 更新 `docs/pipeline/state.yaml`：`phases.2.output = "{文件路径}"`
- 展示设计摘要（架构 + 选型理由）
- 宣布："设计完成。下一阶段：规划。"
- 控制权交还编排器

**如果独立调用：**
- 展示设计摘要
- 建议："设计完成。下一步可调用 /write-plan 创建实施计划"

**终态：** 本技能完成后，唯一的下一步是 `superpowers:writing-plans`。不要调用 frontend-design、code-review 或任何其他实现技能。
