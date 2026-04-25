# 步骤 7：阶段 5 — 规划

**目标**：生成带契约式验收标准的实施计划。

## 执行

调用 `autodev-plan` skill（读取全部 4 个设计文档作为上下文）。

自动模式下，当 writing-plans 提示选择执行方式时，自动选择 "subagent-driven"。

`autodev-plan` 会自动执行：
1. 调用 `superpowers:writing-plans` 生成计划
2. 为每个 task 补充 `acceptance_criteria` + `status: pending`
3. 降阶信号词扫描（红线 3）
4. 依赖版本标注（红线 4）

- **输入**：`*-ideation.md` + `*-design.md` + `*-ui.md` + `*-api.md`
- **产出**：`docs/plans/YYYY-MM-DD-{slug}-plan.md`

> **注意**：Phase 5 完成后，执行 Phase 5.5 生成 index.md 和 rules.md，**然后执行 Plan GAN 审查**（见下文），全部通过后才能进入 Phase 6。Phase 5.5 是 Phase 5 的 POST 步骤的一部分，不需要独立的 state.yaml 条目。

## Plan GAN 审查（代码开发前的方案对抗审查）

<IMPORTANT>
在 Phase 5.5 完成后、进入 Phase 6 之前，**必须**对 plan.md 做一次对抗式方案审查。这是 GAN 3 处的第 1 处：**代码实际开发前，对当前方案进行对抗审查**。

目的：让方案在烧钱写代码之前被独立的 reviewer 挑毛病，避免"计划错了但一路实现下去"。
</IMPORTANT>

### 调用方式

```
Skill(
  skill="autodev-review",
  args="--target plan"
)
```

`autodev-review --target plan` 会自动执行：
1. 收集材料：plan.md + ideation.md + design.md + ui.md + api.md + oss-scan.md
2. Spawn 独立 reviewer agent（不同 context window）
3. 按 4 维度打分：需求覆盖度 / 技术合理性 / 任务可执行性 / 风险与依赖识别
4. 最多 5 轮迭代：NEEDS_IMPROVEMENT 连续 2 轮无提升 → pivot 重写 plan

### 通过条件

- 4 维度全部 ≥ 7 且无 FAIL 项 → PASS，写入 `plan_review_history` 到 plan.md 顶部，进入 Phase 6
- 任一维度 < 5 → FAIL，回到本阶段 `autodev-plan`，按 reviewer SUGGESTIONS 修正 plan.md 后重跑 Plan GAN
- NEEDS_IMPROVEMENT → 修正当前 plan 后再审；5 轮仍未过 → 询问用户是否继续（睡眠模式下默认继续但标注 `plan_review_result: accepted_with_warnings`）

### 失败回退

- 维度 1（需求覆盖度）FAIL → 回 Phase 2（检查能力清单 + design 映射）
- 维度 2（技术合理性）FAIL → 重读 oss-scan.md，考虑是否应该复用开源方案
- 维度 3（任务可执行性）FAIL → 在 Phase 5 重新分解 task
- 维度 4（风险识别）FAIL → 在 plan.md 补风险章节

### 产出物

- `plan.md` 顶部追加 `plan_review_history` 章节（记录每轮打分、最终 VERDICT、reviewer 建议）
- state.yaml 追加字段：`phases.5.plan_review = { verdict, scores, iterations }`

--- 步骤 7 完成 ---
