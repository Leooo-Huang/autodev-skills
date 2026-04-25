# 步骤 8：阶段 5.5 — 生成项目索引和编码规则

**目标**：从设计文档中提取精炼的索引和规则文档，作为开发阶段的高效上下文载体。

## 执行

调用 `autodev-compress` skill。

`autodev-compress` 会自动执行：
1. 读取所有设计文档（ideation/design/ui/api/plan）
2. 生成地图式 `*-index.md`（< 100 行）— 告诉子 agent 信息在哪里
3. 生成 `*-rules.md`（< 80 行）— 编码约束手册

**为什么需要这一步**：开发 subagent 不应加载全部设计文档（容易在长上下文中丢失关键约束）。index 是地图（按需获取），rules 是始终加载的约束。

--- 步骤 8 完成 ---
