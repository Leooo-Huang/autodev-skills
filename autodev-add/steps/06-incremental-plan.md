# 步骤 6：增量实施计划

**目标**：为新功能制定实施计划，考虑与现有代码的集成点。

## 执行

调用 `autodev-plan` skill（增量模式）。

在调用前提供以下上下文：
- 新功能需要修改哪些现有文件
- 新功能需要创建哪些新文件
- 数据库 migration 是否需要
- 哪些改动可能影响现有功能

`autodev-plan` 会自动执行：
1. 调用 `superpowers:writing-plans`
2. 为每个 task 补充 `acceptance_criteria` + `status: pending`
3. 降阶信号词扫描
4. 版本标注

**增量模式特殊行为**：追加到已有 `*-plan.md`（不修改已有任务）。

--- 步骤 6 完成 ---
