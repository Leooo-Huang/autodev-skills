# 步骤 2：Sprint Contract（冲刺契约）

**目标**：为每个待打磨的页面，生成 Generator 和 Evaluator 都认可的验收标准。

> "Before implementation, generator and evaluator negotiate what 'done' means."
> — Anthropic Harness Design

## 为什么需要 Sprint Contract

Sprint Contract 解决的问题：
- Generator 不知道该改到什么程度
- Evaluator 不知道该用什么标准判
- 没有契约 → Generator 乱改，Evaluator 乱判，对抗循环不收敛

## 执行

按 baseline 报告的优先级顺序，逐页面生成 Sprint Contract。

### 对每个页面：

1. **读取 baseline scorecard**：找出该页面每个维度的具体扣分原因

2. **分析 DESIGN.md 与当前代码的 gap**：
   - 用 Grep 搜索页面组件代码中的硬编码颜色值、字体、间距
   - 对比 `polish/design-tokens.md` 中的 token
   - 列出所有不匹配项

3. **生成 Sprint Contract** → 写入 `polish/{page-slug}/sprint-contract.md`：

```markdown
# Sprint Contract: {页面名称}

## Baseline
- 总分：{N}/20
- 最差维度：{维度名} ({分数})
- 截图：{路径}

## Gap 分析
### Compliance 缺陷
- [ ] 主色使用了 #3B82F6 而非 DESIGN.md 的 {正确色值}
- [ ] 标题字体是 Inter 而非 DESIGN.md 的 {正确字体}
- [ ] 按钮圆角 4px 而非 DESIGN.md 的 {正确值}
...

### Craft 缺陷
- [ ] 标题与正文之间间距不一致（24px vs 16px）
- [ ] 卡片阴影过重，与 DESIGN.md 的 elevation 规范不符
...

### Coherence 缺陷
- [ ] 导航栏风格与内容区风格不统一
- [ ] 部分组件用了 shadcn 默认主题，未应用自定义 token
...

## 验收标准
1. 所有 Compliance 缺陷已修复（颜色/字体/间距/圆角/阴影全部匹配 DESIGN.md）
2. CSS 中无硬编码色值——全部通过变量引用
3. Visual Craft ≥ 4（间距精确、层级清晰、对齐无偏差）
4. Coherence ≥ 4（整个页面感觉像同一个设计系统出品）
5. Functionality = 5（所有原有功能正常，无回归）

## PASS 阈值
总分 ≥ 16/20，且无单项 ≤ 2

## 改动边界
- 允许改：CSS / token / 样式变量 / 组件视觉属性 / 动效参数
- 禁止改：页面结构 / 路由 / 数据逻辑 / API 调用 / 业务逻辑
```

4. **契约自检**：
   - 每个缺陷是否具体到可验证？（不能写"改善间距"，要写"标题-正文间距从 24px 改为 DESIGN.md 的 {N}px"）
   - 验收标准是否全部可量化或可截图验证？
   - 改动边界是否明确？

## 批量 vs 逐个

- 如果页面 ≤ 5 个：一次性生成所有 Sprint Contract
- 如果页面 > 5 个：先生成 top 5 低分页面的契约，其余在完成后再生成

## 用户确认（可选）

如果是独立调用（非流水线中），展示第一个页面的 Sprint Contract 给用户确认。用户确认后继续。

流水线中（自动模式）：直接继续到步骤 3。

--- 步骤 2 完成 ---
