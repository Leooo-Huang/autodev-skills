---
name: autodev-sync
description: "文档进度同步：读取代码和 git 历史，更新设计文档中的完成状态。当用户说 同步文档、更新进度、打勾、sync 时触发。"
user-invocable: true
allowed-tools: [Bash, Read, Write, Glob, Grep, TodoWrite]
---

# AutoDev Sync — 文档进度同步

读取代码实现和 git 历史，回到设计文档中标注哪些功能已完成、哪些正在进行、哪些还没开始。核心原则：**文档是活的，不是写完就丢的。**

## 何时使用

- 一批功能开发完，需要更新文档状态
- 用户说"同步文档"、"更新进度"、"哪些做了哪些没做"
- MVP 完成后，准备开始 V2，需要知道当前进度

何时不使用：
- 需要生成全新的文档 → `/autodev-reverse`
- 需要修改功能设计本身 → 手动编辑或重跑对应阶段

## 步骤索引

| 步骤 | 文件 | 说明 |
|------|------|------|
| 1 | `steps/01-scan-docs.md` | 扫描设计文档，提取可追踪条目 |
| 2 | `steps/02-scan-code.md` | 扫描代码实现，判断每项完成状态 |
| 2.5 | `steps/03-design-drift.md` | 设计漂移检测（技术选型/API/数据模型） |
| 3 | `steps/04-compare-diff.md` | 对比文档与代码的差异 |
| 4 | `steps/05-update-docs.md` | 在原文档中就地更新状态标记 |
| 5 | `steps/06-generate-report.md` | 生成同步报告 |

**辅助资源：**
- `templates/sync-report.md` — 同步报告输出模板
- `checklists/stub-detection.md` — 桩代码检测清单
- `checklists/drift-detection.md` — 漂移检测清单

<IMPORTANT>
按步骤顺序执行。执行步骤 N 时，用 Read 工具读取对应的 steps/ 文件获取详细指令。
只加载当前步骤的文件，不要一次性读取所有步骤——这是为了节省 token 并保持专注。
每步完成后输出 `--- 步骤 N 完成 ---`。
共享资源在 `autodev-shared/` 目录下，按需读取。
</IMPORTANT>

## 批次追踪

从 `$ARGUMENTS` 解析批次范围。无参数时同步全部。
- `/autodev-sync` — 同步全部文档
- `/autodev-sync mvp` — 只同步 MVP 范围
- `/autodev-sync v2` — 只同步 V2 范围

## 反模式

| 禁止 | 应该 |
|------|------|
| 修改功能描述或设计内容 | 只更新状态标记 |
| 猜测实现状态 | 基于代码搜索和 git 历史判断 |
| 删除未实现的功能条目 | 保留所有条目，标记状态 |
| 创建新的进度文档替代原文档 | 在原文档上就地更新 |
| 不做桩代码扫描就标状态 | 先扫描桩代码信号，再判断状态 |
| 核心数据管道断裂仍标 `[~]` | 数据源为空/桩/未接通 = `[ ]` 未开始 |
| 只检查完成状态不检查设计漂移 | 步骤 2.5 检测技术选型、API、数据模型是否与文档一致 |
