# 步骤 6：阶段 4 — API 设计

**目标**：调用 `autodev-api` 技能，从 UI 数据需求倒推 API 端点。

## 执行

调用 `autodev-api` 技能。

**衔接：** 该技能会自动读取 `*-design.md`（数据模型、架构）和 `*-ui.md`（页面数据需求汇总）作为输入上下文。

- **输入**：`*-design.md` + `*-ui.md`
- **产出**：`docs/plans/YYYY-MM-DD-{slug}-api.md`

## 验证

产出文件必须包含"API 端点"章节。

--- 步骤 6 完成 ---
