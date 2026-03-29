---
name: autodev-iterate
description: "迭代开发：对已有功能做 bug 修复、改进、增量更新。当用户说 iterate、迭代、修复、改进、优化已有功能、fix、improve 时触发。不适用于从零开发新功能（用 /autodev）。"
user-invocable: true
allowed-tools:
  - Agent
  - Bash
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Skill
  - TodoWrite
  - AskUserQuestion
  - EnterPlanMode
  - ExitPlanMode
  - TeamCreate
  - SendMessage
  - TeamDelete
---

# AutoDev Iterate — 迭代开发

对**已有功能**做 bug 修复、改进、增量更新。核心原则：**先思考，后执行。** 不是看到问题就改代码，而是先做全面分析——UX、技术、代码三个维度并行研究，综合后再设计方案。

## 何时使用

- 已有功能不工作（bug 修复）、需要改进（性能、准确率、交互优化）
- 用户说"迭代"、"修复"、"改进"、"优化"、"fix"、"improve"

何时不使用：
- 从零开发全新功能 → `/autodev`
- 只需同步文档状态 → `/autodev-sync`
- 从代码反向生成文档 → `/autodev-reverse`

## 步骤索引

| 阶段 | 文件 | 说明 |
|------|------|------|
| 0 | `steps/00-multi-iterate-orchestration.md` | 多条迭代编排（>=2 条修改时） |
| 1 | `steps/01-read-docs.md` | 读文档，理解设计意图，定位用户旅程 |
| 2 | `steps/02-create-research-team.md` | 创建研究团队（UX/技术/代码三路并行） |
| 3 | `steps/03-synthesize.md` | 综合分析，合并三路结论 |
| 4 | `steps/04-design-solution.md` | 设计方案（plan mode） |
| 5 | `steps/05-implement.md` | 实现 |
| 6 | `steps/06-verify.md` | 验证（静态+运行时+旅程连续性） |
| 7 | `steps/07-sync-and-fix-docs.md` | 同步文档 + 设计修正（强制） |

**辅助资源：**
- `checklists/ux-audit.md` — UX 审查清单
- `checklists/journey-continuity.md` — 旅程连续性检查

<IMPORTANT>
按步骤顺序执行。执行步骤 N 时，用 Read 工具读取对应的 steps/ 文件获取详细指令。
只加载当前步骤的文件，不要一次性读取所有步骤——这是为了节省 token 并保持专注。
每步完成后输出 `--- 阶段 N 完成 ---`。
共享资源在 `autodev-shared/` 目录下，按需读取。

**绝对不可跳过任何阶段。** 无论改动看起来多简单（哪怕只改一行 CSS），都必须按顺序执行阶段 1-7。
- **阶段 2**：必须用 Agent 工具创建至少 2 个并行研究代理（涉及 UI 时 3 个）。**不允许自己做分析替代研究团队**。
- **阶段 4**：必须向用户展示方案。**不允许直接跳到实现**。
- **违反检测**：如果你在没有调用 Agent 工具的情况下就开始 Edit/Write 代码文件，你就跳过了阶段 2-3，必须停下来回到阶段 2。
</IMPORTANT>

## 反模式（精简版）

| 禁止 | 应该 |
|------|------|
| 不读文档直接改代码 | 先理解设计意图 |
| 自己一个人做所有分析 | 创建研究团队并行分析 |
| 想到一个方案就直接做 | 至少评估 2 种方案再选 |
| 只跑 tsc/pytest 就宣布修好了 | 必须 curl API 或直接调函数验证实际输出 |
| 多条修改合并在一起一次性做 | 逐条走完整 iterate 流程 |
| 实现偏离设计文档但不修正文档 | 阶段 7a 主动修正设计文档 |
