# 步骤 10：阶段 7 — 验证

**目标**：调用 `autodev-verify` 执行统一验收。

## 执行

调用 `autodev-verify` skill（全部 5 层验证）。

`autodev-verify` 会自动执行：
1. **契约验收**：对照 plan.md 的 acceptance_criteria + reviewer 记录
2. **红线扫描**：占位/Mock/降阶/版本 最终确认（PostToolUse hook 应已拦截大部分）
3. **静态检查**：tsc / ruff / pytest 等
4. **运行时验证**：API 端点调用 + UI 流程检查
5. **acceptance-testing**：端到端功能验证

- **输入**：已提交的代码 + `*-ui.md` + `*-api.md` + plan.md（acceptance_criteria）
- **产出**：验收报告（通过/需修复）
- **阻塞规则**：如果验证未通过，不得进入 Global GAN（下一步）。失败时回到阶段 6 修复代码，再重新验证。

## Global GAN 审查（全部完成后的全局对抗审查）

<IMPORTANT>
`autodev-verify` 通过后，**必须**对整个项目做一次全局对抗审查。这是 GAN 3 处的第 3 处：**全部完成后，对全局进行对抗审查**。

目的：跳出单 task / 单页面视角，从"整个交付物是否兑现了 ideation 和 design"的视角独立评估。verify 通过 ≠ 交付质量达标——verify 查的是契约逐条、红线扫描；global GAN 查的是端到端可用、架构一致、需求对齐。
</IMPORTANT>

### 调用方式

```
Skill(
  skill="autodev-review",
  args="--target global"
)
```

`autodev-review --target global` 会自动执行：
1. 收集全局材料：
   - `git log main..HEAD --oneline` + `git diff main...HEAD --stat`
   - 全部设计文档（ideation/design/ui/api/plan/rules/index）
   - `docs/pipeline/state.yaml`（所有阶段完成状态）
   - `autodev-verify` 产出的验收报告
2. Spawn 独立全局 reviewer agent
3. 按 4 维度打分：
   - 需求对齐（ideation MVP 全覆盖）
   - 架构一致（符合 design.md 组件划分）
   - 端到端可用（核心用户故事走通）
   - 文档-代码对齐（rules.md 约束遵守）
4. 最多 3 轮迭代（全局修复代价高）

### 通过条件

- 4 维度全部 ≥ 7 → PASS → 写 `docs/pipeline/global-review.md`，进入 Phase 8
- 任一维度 < 5 → FAIL，按失败维度回到对应阶段：
  - 维度 1 FAIL → 回 Phase 1/2（补需求 + 回跑后续）
  - 维度 2 FAIL → 回 Phase 2（改架构 + 回跑 Phase 5 Plan GAN）
  - 维度 3 FAIL → 回 Phase 6（修代码，针对断链的用户故事）
  - 维度 4 FAIL → 回 Phase 5.5（补 rules）或 Phase 6（修代码遵守 rules）
- NEEDS_IMPROVEMENT → 按 SUGGESTIONS 修复后重跑 Global GAN
- 3 轮仍未过 → 睡眠模式下继续 Phase 8 但标注 `global_review_result: accepted_with_warnings`，非睡眠模式下询问用户

### 产出物

- `docs/pipeline/global-review.md`（含 4 维度打分、reviewer 建议、最终 VERDICT）
- state.yaml 追加字段：`phases.7.global_review = { verdict, scores, iterations }`

**阻塞规则**：Global GAN 未 PASS（或 3 轮接受警告），不得进入 Phase 8 交付。

--- 步骤 10 完成 ---
