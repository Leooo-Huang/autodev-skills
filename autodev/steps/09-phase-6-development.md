# 步骤 9：阶段 6 — 开发

**目标**：调用 `superpowers:subagent-driven-development` 执行编码，确保每个 subagent 携带精炼上下文和质量红线。

## 执行

调用 `superpowers:subagent-driven-development` 技能，读取实施计划。
如果 `use_worktree: true` 且尚未在 worktree 中，先创建 worktree。

## 上下文加载策略（Token 效率优先）

每个 subagent 的任务描述中，必须包含以下精炼上下文：
1. `*-index.md` 全文（< 100 行，项目全景）
2. `*-rules.md` 全文（< 80 行，编码约束）
3. `*-plan.md` 中**当前任务**对应的章节（不是全文）

**按需加载**：仅当 subagent 需要某个模块的详细设计时，在任务描述中指示它读取对应文档：
- 需要页面布局详情 → 读 `*-ui.md` 对应页面章节
- 需要 API 接口详情 → 读 `*-api.md` 对应端点章节
- 需要架构决策理由 → 读 `*-design.md` 对应章节

**禁止**：不要把全部设计文档塞进 subagent 的初始 prompt。

## 质量红线传递（必须传给每个 subagent）

<IMPORTANT>
每个 subagent 的任务描述中，必须附加以下质量约束：

```
质量红线（违反任何一条则任务未完成）：
1. 禁止占位：不允许 TODO、空函数体、pass、NotImplementedError。功能在范围内就必须完整实现。
2. 禁止 Mock：不允许 mock/dummy/fake/placeholder 数据替代真实实现。需要调 API 就调 API。
3. 禁止降阶：按设计文档指定的技术方案实现，不得替换为"更简单的替代方案"。
4. 版本正确：安装依赖时使用设计文档技术选型表中标注的版本号，不要凭记忆用旧版本。
   遇到不确定的 API 用法 → WebSearch 查当前版本文档，不要猜。
```

不传递质量约束的 subagent 调度视为阶段未完成。
</IMPORTANT>

## 前端代码质量规则

当任务涉及前端页面或组件时（判断依据：`*-ui.md` 存在且该任务对应 ui.md 中的某个页面），必须：

1. **传入 ui.md 作为结构约束**：subagent 在实现前端页面时，必须读取 `*-ui.md` 中该页面的布局区域、组件清单、状态设计
2. **通过 `frontend-design` skill 生成 UI 代码**：所有用户可见的页面和组件，必须调用 `frontend-design` skill 来生成代码
3. **读取前端技术栈偏好**：检查 `config.yaml` 的 `phases.frontend` 配置（如有）

**无前端的项目（CLI、纯 API）**：跳过以上规则。判断依据：`*-ui.md` 不存在或 `config.yaml` 中 `frontend.stack: "none"`。

- **输入**：`*-plan.md` + `*-ui.md`（前端任务时）
- **产出**：已提交的代码

--- 步骤 9 完成 ---
