---
name: autodev-brainstorm
description: "调研完成后设计功能：发散出多种方案、评估、收敛到最佳设计。当需要以调研为基础进行设计探索时使用。当用户说 autodev-brainstorm、头脑风暴、设计方案 时触发。"
user-invocable: true
allowed-tools: [Bash, Read, Write, Skill, TodoWrite]
---

# AutoDev Brainstorm — 第 2 阶段：头脑风暴

你是技术设计专家。基于调研成果，通过发散、评估、收敛的循环，产出一份经过验证的设计文档。

核心原则：**调研驱动的发散思维产出更好的设计。**

## 何时使用

- AutoDev 流水线的第 2 阶段（由编排器调度）
- 独立使用：有调研结果，想探索设计方案
- 用户说"设计方案"、"头脑风暴"、"brainstorm"

何时不使用：
- 没有调研且想要交互式头脑风暴 → 用 `superpowers:brainstorming`
- 设计已确定，只需实施计划 → 用 `superpowers:writing-plans`

## 步骤索引

| 步骤 | 文件 | 说明 |
|------|------|------|
| 1 | `steps/01-load-input.md` | 加载调研文档 + 项目上下文 |
| 2 | `steps/02-diverge.md` | DIVERGE：生成 3-5 个架构层面本质不同的方案 |
| 3 | `steps/03-evaluate.md` | EVALUATE：4 维度评分矩阵 |
| 4 | `steps/04-converge.md` | CONVERGE：自动/门控选择最佳方案 |
| 5 | `steps/05-design.md` | DESIGN：详细设计 + 技术鲜度验证 |
| 6 | `steps/06-self-check.md` | SELF-CHECK（→ `checklists/self-check.md`） |
| 7 | `steps/07-write-document.md` | 写入设计文档（→ `templates/design-output.md`） |
| 8 | `steps/08-handoff.md` | 交接 writing-plans |

## 执行规则

<IMPORTANT>
按步骤顺序执行。执行步骤 N 时，用 Read 工具读取对应的 steps/ 文件获取详细指令。
只加载当前步骤的文件，不要一次性读取所有步骤——这是为了节省 token 并保持专注。
每步完成后输出 `--- 步骤 N 完成 ---`。
共享资源在 `autodev-shared/` 目录下，按需读取。
</IMPORTANT>

<HARD-GATE>
在设计文档写入并通过自检之前，绝不调用 writing-plans 或任何实现技能。
</HARD-GATE>

## 反模式（精简版）

| 禁止 | 应该 |
|------|------|
| "方案"只是换个框架名字 | 每个方案在架构层面有本质差异 |
| 评分时忽略调研发现 | 调研匹配度是评分维度之一 |
| 跳过评分矩阵直接选方案 | 即使自动模式也走评分流程 |
| 设计文档没有"风险与缓解" | 调研中的陷阱必须在设计中有对策 |
| 没有调研就照常跑全流程 | 至少警告用户"无调研支撑" |
