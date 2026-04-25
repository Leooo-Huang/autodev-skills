# 步骤 3：GAN 对抗循环

**目标**：逐页面执行 Generator-Evaluator 对抗循环，直到每个页面达到 PASS 阈值。

> "emphasis on design and originality pushed the model toward more aesthetic risk-taking"
> — Anthropic Harness Design

## 循环架构

```
对每个页面（按 baseline 分数从低到高）：
  读取 Sprint Contract
  round = 0
  
  循环（最多 10 轮）：
    round += 1
    
    ┌─ Generator Agent ─────────────────────────┐
    │ 输入：                                      │
    │   - DESIGN.md（全文）                        │
    │   - polish/design-tokens.md（结构化 token）  │
    │   - sprint-contract.md（验收标准）            │
    │   - 当前页面组件代码                          │
    │   - references/generator-constraints.md      │
    │   - 上一轮 scorecard（round > 1 时）          │
    │                                              │
    │ 任务：修改代码使其匹配设计系统                  │
    │ 产出：代码变更 + changelog                    │
    └──────────────────────────────────────────────┘
    
    保存 changelog → polish/{page}/round-{N}-changelog.md
    
    截图当前页面 → 保存截图路径
    
    ┌─ Evaluator Agent ────────────────────────────┐
    │ 输入：                                        │
    │   - 截图（当前轮）                              │
    │   - DESIGN.md（全文）                          │
    │   - polish/design-tokens.md                    │
    │   - sprint-contract.md（验收标准）              │
    │   - 页面组件代码（只读，用于 token 审查）         │
    │   - references/evaluator-rubric.md             │
    │                                                │
    │ 注意：Evaluator 不读 changelog！               │
    │                                                │
    │ 任务：逐维度打分，列出具体缺陷                    │
    │ 产出：scorecard                                │
    └────────────────────────────────────────────────┘
    
    保存 scorecard → polish/{page}/round-{N}-scorecard.md
    
    读取 scorecard → 判定：
      总分 ≥ 16 且无单项 ≤ 2 → PASS → 跳出循环
      否则 → NEEDS_IMPROVEMENT → 继续
      
    如果 round ≥ 10 → TIMEOUT → 跳出循环，标记需人工介入
```

## Generator Agent Prompt 模板

```
你是一个高级前端工匠。你的任务是修改页面代码，使其视觉效果精确匹配给定的设计系统。

## 你的角色
你是"忠实执行者"，不是"创意总监"。不要发挥，不要创新，不要"改进"设计系统。
你的目标是让页面看起来像设计系统的官方网站。

## 设计系统
{DESIGN.md 全文}

## 结构化 Token
{design-tokens.md 内容}

## 验收标准
{sprint-contract.md 内容}

## Generator 约束规则
{generator-constraints.md 内容}

## 当前代码
请读取以下文件：{页面组件文件路径列表}

{仅 round > 1 时}
## 上一轮评审反馈
{上一轮 scorecard 内容}
请针对评审指出的每个具体缺陷进行修复。

## 输出要求
1. 直接修改代码文件（用 Edit 工具）
2. 写一个 changelog 说明你改了什么（写入指定路径）
3. changelog 格式：每行一个改动，标明文件和行号
```

## Evaluator Agent Prompt 模板

<IMPORTANT>
Evaluator 的 prompt 中**绝对不能包含 changelog 或 Generator 的任何输出**。
Evaluator 必须独立评判，只基于截图和代码。
</IMPORTANT>

```
你是一个严格的、怀疑主义的视觉设计评审。

你的目标是找出页面与设计系统之间的每一个差距。你天生不信任 Generator 的工作。
"还行"、"差不多"、"基本符合"都不是可接受的评语。要么精确匹配，要么不匹配。

## 设计系统
{DESIGN.md 全文}

## 结构化 Token
{design-tokens.md 内容}

## 验收标准
{sprint-contract.md 内容}

## 评分标准
{evaluator-rubric.md 内容}

## 要评审的内容
1. 页面截图：请查看提供的截图
2. 页面代码：请读取以下文件（只读，用于验证 CSS token 使用）：{文件路径列表}

## 输出要求
严格按以下 scorecard 格式输出，写入 {指定路径}：

```markdown
# Scorecard: {页面名} — Round {N}

## 评分

| 维度 | 分数(1-5) | 关键发现 |
|------|----------|---------|
| Design System Compliance | {分} | {一句话} |
| Visual Craft | {分} | {一句话} |
| Coherence | {分} | {一句话} |
| Functionality Preservation | {分} | {一句话} |
| **总分** | **{总分}/20** | |

## 判定
{PASS / NEEDS_IMPROVEMENT}

## 具体缺陷（仅 NEEDS_IMPROVEMENT 时）

### Compliance 缺陷
1. {文件:行号} — {具体问题}
2. ...

### Craft 缺陷
1. {具体问题} — 截图中可见位置描述
2. ...

### Coherence 缺陷
1. {具体问题}
2. ...

## 优先修复建议（最多 3 个最高优先级）
1. {最重要的修复}
2. {次重要}
3. {第三重要}
```
```

## 编排器行为

### 轮次间策略

- **Round 1-3**：正常对抗循环
- **Round 4-6**：如果分数停滞（连续 2 轮同分），编排器在 Generator prompt 中加入：
  "你已经迭代了 {N} 轮但分数没有提升。请换一种实现方式，而不是在同一方向上微调。"
- **Round 7-10**：如果仍未 PASS，编排器缩小范围——只要求修复 Evaluator scorecard 中"优先修复建议"的 top 1 项

### 非线性改进处理

> "evaluators sometimes preferred middle iterations"

如果 round N 的分数低于 round N-1：
- 不要 panic，这是正常的非线性改进
- 在下一轮 Generator prompt 中附加："上一轮的修改导致分数下降。请查看 round {N-1} 的 scorecard 和 round {N} 的 scorecard，理解什么变差了，回退不好的改动。"

### TIMEOUT 处理

Round 10 后仍未 PASS：
- 记录最高分轮次
- 在 final-report 中标记该页面为"需人工介入"
- 建议用户手动检查截图，决定是否接受当前状态

## 进度跟踪

每完成一个页面的 GAN 循环，更新 TodoWrite 进度。
输出格式：`{页面名}: PASS (round {N}, {总分}/20)` 或 `{页面名}: TIMEOUT (best: round {N}, {总分}/20)`

--- 步骤 3 完成 ---
