# 步骤 4：Final Sweep（终审扫描）

**目标**：全站截图对比 before/after，生成总结报告，确认无回归。

## 4.1 全站重新截图

对所有页面重新截图（包括 PASS 和 TIMEOUT 的）。

## 4.2 功能回归检查

对每个页面做基本交互验证：
- 页面能正常加载（无白屏、无控制台报错）
- 主要交互元素可点击（按钮、链接、表单）
- 如果有 playwright MCP：运行 `browser_snapshot` 获取可访问性树，确认关键元素存在

如果发现功能回归：
1. 记录回归问题
2. Spawn 一个修复 agent 恢复功能
3. 重新截图验证

## 4.3 生成 Final Report

写入 `polish/final-report.md`：

```markdown
# AutoDev Polish — Final Report

## 概览
- 设计系统：{DESIGN.md 来源/品牌名}
- 打磨页面数：{N}
- 总耗时：{估算}
- 总轮次：{所有页面的轮次之和}

## 页面结果

| 页面 | Baseline | Final | 轮次 | 状态 |
|------|---------|-------|------|------|
| {页面名} | {N}/20 | {N}/20 | {N} | PASS / TIMEOUT |
...

## 整体提升
- 平均分：{before} → {after} (+{delta})
- PASS 率：{N}/{total}

## 未解决问题（TIMEOUT 页面）
- {页面名}：{最后一轮 Evaluator 的主要缺陷}
...

## 功能回归检查
- {PASS / 发现 N 个问题已修复 / 发现 N 个问题需人工处理}
```

## 4.4 展示给用户

输出 final report 摘要，包括：
- 总体提升幅度
- 每个页面的 before/after 分数
- 需要人工关注的问题（如有）

--- 步骤 4 完成 ---
