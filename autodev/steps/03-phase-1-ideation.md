# 步骤 3：阶段 1 — 产品创意

**目标**：调用 `autodev-ideation` 技能，从用户想法中提炼产品创意和 MVP 功能清单。

## 执行

调用 `autodev-ideation` 技能，传递主题和用户的初始思路。

- **输入**：用户的想法/文档
- **产出**：`docs/plans/YYYY-MM-DD-{slug}-ideation.md`

## 衔接

ideation 技能会自动进行：
1. 第一性原理拆解（核心价值链分析）
2. 调研-创意循环（最多 `config.phases.ideation.max_iterations` 次迭代）
3. 产出 MVP 功能清单（含平台层 P 编号 + 场景层 W 编号）

## 验证

产出文件必须包含：
- "MVP" 章节
- 核心价值链分析
- 失败模式分析（Q3）

--- 步骤 3 完成 ---
