---
name: autodev-polish
description: "当项目 UI 视觉效果不佳、需要匹配设计系统（DESIGN.md）、或需要系统性视觉升级时使用。当用户说 polish、视觉升级、UI打磨、页面太丑、redesign 时触发。"
user-invocable: true
allowed-tools: [Read, Write, Edit, Glob, Grep, Bash, Agent, WebFetch, WebSearch, TodoWrite, AskUserQuestion, TaskCreate, TaskUpdate]
---

# AutoDev Polish — GAN 式视觉打磨

用 Generator-Evaluator 对抗循环，将项目 UI 逐页面打磨至专业设计系统水准。

灵感来源：[Anthropic Harness Design](https://www.anthropic.com/engineering/harness-design-long-running-apps) — "tuning a standalone evaluator to be skeptical turns out to be far more tractable than making generators self-critical."

## 何时使用

- 项目 UI 效果不佳，需要系统性视觉提升
- 想让 UI 匹配某个知名品牌的设计系统（Stripe / Linear / Vercel 等）
- autodev / autodev-add 产出的前端页面需要打磨

何时不使用：
- 没有可运行的前端页面 → 先 `/autodev`
- 只改一个组件样式 → `/autodev-iterate`
- 需要重新设计页面结构 → `/autodev-ui`

## 前置条件

1. **可运行的前端**（dev server 能启动）
2. **DESIGN.md**：项目根目录的视觉真值标准（步骤 1 会引导获取）
3. **截图能力**：chrome-devtools 或 playwright MCP 至少一个可用

## 步骤索引

| 步骤 | 文件 | 说明 |
|------|------|------|
| 1 | `steps/01-load-and-audit.md` | 加载 DESIGN.md + 截图全站 + baseline 评分 |
| 2 | `steps/02-sprint-contract.md` | 每页面协商验收契约 |
| 3 | `steps/03-gan-loop.md` | Generator 改 → 截图 → Evaluator 判 → 迭代 |
| 4 | `steps/04-final-sweep.md` | 全站 before/after 对比 + 总结报告 |

## 执行规则

<IMPORTANT>
按步骤顺序执行。执行步骤 N 时，用 Read 工具读取对应的 steps/ 文件。
只加载当前步骤，不要一次性读取所有步骤。
每步完成后输出 `--- 步骤 N 完成 ---`。

架构铁律：
1. Generator 和 Evaluator 必须是**独立 agent**（Agent 工具 spawn），不共享上下文
2. Evaluator **不知道** Generator 改了什么——只看截图和代码，拿 DESIGN.md 当尺子
3. 所有 agent 间通信**通过文件**，不通过对话
4. Evaluator 评分标准在 `references/evaluator-rubric.md`——措辞经过设计，直接影响收敛方向
5. Generator 约束规则在 `references/generator-constraints.md`
</IMPORTANT>

## 产出目录

```
polish/                           # 打磨工作目录（项目根下）
  design-tokens.md                # 从 DESIGN.md 提取的结构化 token
  baseline-report.md              # 初始评估报告
  {page-slug}/                    # 每个页面
    sprint-contract.md            # 验收契约
    round-{N}-changelog.md        # Generator 改动说明
    round-{N}-scorecard.md        # Evaluator 评分卡
  final-report.md                 # before/after 对比报告
```

## 反模式

| 禁止 | 应该 |
|------|------|
| Generator 自评自己的改动 | 独立 Evaluator 评判 |
| Evaluator 看 Generator 的 changelog | Evaluator 只看截图和代码 |
| 一次改所有页面 | 逐页面迭代，达标后下一页 |
| 改功能代码或页面结构 | 只改样式/token/视觉代码 |
| 跳过截图直接评分 | 每轮必须截图 |
| Evaluator 宽容（"还行"） | 严格按 rubric 高标准打分 |
