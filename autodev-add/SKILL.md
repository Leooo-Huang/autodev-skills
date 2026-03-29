---
name: autodev-add
description: "增量功能开发：给已有项目添加新功能。读取现有设计文档，只对新功能跑 ideation→design→ui→api→plan 增量流程，然后实现和验证。当用户说 add、新增功能、加一个功能 时触发。"
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
---

# AutoDev Add — 增量功能开发

给**已有项目**添加新功能。与 `/autodev` 的区别：autodev 从零生成全部设计，autodev-add 在现有设计基础上只做增量。

## 何时使用

- 项目已有设计文档和代码，想加一个新功能
- 用户说"加一个功能"、"新增 X"、"add feature"
- 已有项目需要扩展新模块

何时不使用：
- 从零开始 → `/autodev`
- 修复/改进已有功能 → `/autodev-iterate`
- 只需同步文档 → `/autodev-sync`

## 步骤索引

| 步骤 | 文件 | 说明 |
|------|------|------|
| 1 | `steps/01-read-existing.md` | 读取现有设计文档，理解当前架构和功能 |
| 2 | `steps/02-incremental-ideation.md` | 只对新功能做创意分析（不重复已有功能） |
| 3 | `steps/03-incremental-design.md` | 新功能的技术设计（融入现有架构） |
| 4 | `steps/04-incremental-ui.md` | 新功能的 UI 设计（与现有页面协调） |
| 5 | `steps/05-incremental-api.md` | 新功能的 API 设计（复用现有数据模型） |
| 6 | `steps/06-incremental-plan.md` | 新功能的实施计划 |
| 7 | `steps/07-update-spec.md` | 更新 index.md + rules.md |
| 8 | `steps/08-implement.md` | 实现新功能 |
| 9 | `steps/09-verify.md` | 验证（新功能 + 不破坏已有功能） |
| 10 | `steps/10-sync-docs.md` | 同步文档状态 |

<IMPORTANT>
按步骤顺序执行。执行步骤 N 时，用 Read 工具读取对应的 steps/ 文件获取详细指令。
只加载当前步骤的文件，不要一次性读取所有步骤——这是为了节省 token 并保持专注。
每步完成后输出 `--- 步骤 N 完成 ---`。
共享资源在 `autodev-shared/` 目录下，按需读取。
</IMPORTANT>

## 反模式

| 禁止 | 应该 |
|------|------|
| 重新生成已有功能的设计 | 只对新功能做增量设计 |
| 忽略现有架构直接设计 | 先理解现有架构，在其上扩展 |
| 新功能破坏已有功能 | 验证阶段必须检查回归 |
| 追加的设计和现有文档风格不一致 | 匹配现有文档的格式和术语 |
| 不更新 index.md/rules.md | 新功能加入后必须更新摘要 |
