# 步骤 9：验证新功能

**目标**：验证新功能正常工作且不影响已有功能。

## 执行

调用 `autodev-verify` skill（全部 5 层验证）。

`autodev-verify` 会自动执行：
1. **契约验收**：对照 plan.md 新增 task 的 acceptance_criteria
2. **红线扫描**：占位/Mock/降阶/版本
3. **静态检查**：tsc / ruff 等
4. **运行时验证**：新功能 API + 已有功能回归
5. **acceptance-testing**：端到端验证

**回归特别注意**：新功能不能破坏已有功能。如果验证发现回归 → 回退到步骤 8 修复。

--- 步骤 9 完成 ---
