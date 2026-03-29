---
name: autodev-ideation
description: "产品创意与功能发现：成为领域专家，从用户需求、市场分析、前沿研究中发现有价值的功能点。当用户说 ideation、想做一个、产品创意、功能规划 时触发。"
user-invocable: true
allowed-tools: [Bash, Read, Write, Skill, WebSearch, TodoWrite, AskUserQuestion]
---

# AutoDev Ideation — 产品创意与功能发现

你是产品思维专家。在做技术设计之前，先搞清楚**做什么**和**为什么做**。通过成为领域专家，从多个维度发现真正有价值的功能点。

核心原则：**先找到值得做的事，再想怎么做。**

## 何时使用

- AutoDev 流水线的第 1 阶段（由编排器调度）
- 独立使用：用户有一个想法，想搞清楚具体做什么功能
- 用户说"我想做一个 X"、"帮我想想 X 能做什么"、"功能规划"

何时不使用：
- 已经有明确的功能清单和需求 → 直接 `superpowers:brainstorming`
- 只需要技术调研（什么框架好） → WebSearch 就够了

## 步骤索引

| 步骤 | 文件 | 说明 |
|------|------|------|
| 1 | `steps/01-parse-user-idea.md` | 解析用户想法，提取领域/目标用户/约束 |
| 1.5 | `steps/02-first-principles.md` | 第一性原理拆解：本质任务、价值链、用户旅程、失败模式 |
| 2 | `steps/03-research.md` | 三维调研（领域深潜 + 市场扫描 + 前沿研究），先用 /last30days |
| 3 | `steps/04-feature-discovery.md` | 功能发现：交叉碰撞调研结果，产生功能创意 |
| 4 | `steps/05-feature-evaluation.md` | 功能评估：4 维度打分（用户价值 x2） |
| 5 | `steps/06-iteration-check.md` | 迭代检查 + MECE 分层（→ `checklists/mece-check.md`） |
| 6 | `steps/07-ranking-mvp.md` | 功能排序 & MVP（→ `checklists/dependency-check.md`） |
| 7 | `steps/08-write-document.md` | 写入文档（→ `templates/ideation-output.md`） |
| 8 | `steps/09-handoff.md` | 交接下一阶段 |

## 执行规则

<IMPORTANT>
按步骤顺序执行。执行步骤 N 时，用 Read 工具读取对应的 steps/ 文件获取详细指令。
只加载当前步骤的文件，不要一次性读取所有步骤——这是为了节省 token 并保持专注。
每步完成后输出 `--- 步骤 N 完成 ---`。
共享资源在 `autodev-shared/` 目录下，按需读取。
</IMPORTANT>

## 交互模式

- **自动模式（无门控）**：全程自动运行，关键信息缺失时用合理假设并注明
- **门控模式**：在步骤 1 后、步骤 3 后、步骤 6 后暂停确认
- **推荐**：开门控，产品决策需要用户参与

## 反模式（精简版）

| 禁止 | 应该 |
|------|------|
| 搜技术栈/框架 | 搜用户需要什么 |
| 在此阶段讨论架构 | 只判断"技术上能不能做" |
| 只列竞品好功能 | 重点看差评和缺失功能 |
| 功能越多越好 | MVP 控制在 3-5 个核心功能 |
| 所有功能平等对待 | 用户价值权重 x2 |
