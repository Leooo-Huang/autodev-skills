---
name: autodev-review
description: GAN 式对抗代码审查：spawn 独立怀疑主义 reviewer，4 维度打分，迭代改进或 pivot。可独立使用，也被 autodev/autodev-add/autodev-iterate 自动调用。
metadata:
  promptSignals:
    phrases:
      - "autodev-review"
      - "代码审查"
      - "GAN审查"
      - "对抗审查"
      - "review代码"
---

# AutoDev Review — GAN 式对抗代码审查

基于 Anthropic 研究（Harness Design for Long-Running Apps）：agent 评价自己的输出时倾向于"自信地称赞"，分离生成和评估是最有效的解决方法。

## 使用场景

- **被编排器调用**：autodev Phase 6 / autodev-add Step 8 / autodev-iterate Step 5 自动调用
- **独立使用**：`/autodev-review` 随时对当前代码变更做对抗审查

## 审查对象（--target 参数）

`autodev-review` 支持 4 种审查对象，每种对应不同的输入材料、4 维度、失败回退路径。默认 `code`。

| target | 时机（autodev 流水线） | 输入材料 | 4 维度 | 失败回退 |
|--------|---------------------|---------|--------|---------|
| `code`（默认） | Phase 6 每个 code task 完成后 | `git diff` + plan.md task 的 acceptance_criteria + rules.md | ① 功能完整性 ② 健壮性 ③ 设计一致性 ④ 代码质量 | 继续修改当前 task 代码 |
| `plan` | Phase 5 完成后、进入 Phase 6 前 | `*-plan.md` + `*-ideation.md` + `*-design.md` + `*-ui.md` + `*-api.md` | ① 需求覆盖度 ② 技术合理性 ③ 任务可执行性 ④ 风险与依赖识别 | 回 Phase 5 修改 plan |
| `ui` | Phase 6 每个 UI/前端 task 完成后 | UI 代码 `git diff` + `*-ui.md` 对应页面章节 + 浏览器渲染截图/DOM 快照 | ① 视觉层级与信息密度 ② 交互流畅度 ③ 状态完整性（loading/空/错误/成功）④ 与 ui.md 一致性 | 回 Phase 6 该 UI task |
| `global` | Phase 7 验证通过后、Phase 8 交付前 | 全部代码（`git diff main...HEAD`）+ 全部设计文档 + verify 报告 | ① 需求对齐（ideation MVP 功能全覆盖）② 架构一致（符合 design.md 组件划分）③ 端到端可用（核心用户故事走通）④ 文档-代码对齐（rules.md 约束遵守） | 按失败维度回对应阶段 |

调用方式：

```
/autodev-review                    # 默认 target=code
/autodev-review --target plan
/autodev-review --target ui
/autodev-review --target global
```

## 输入

自动检测（按优先级）：
1. 如果提供了具体 acceptance_criteria → 用它
2. 如果 `*-plan.md` 存在且当前 task 有 acceptance_criteria → 读取
3. 如果都没有 → 从 git diff 推断审查标准（功能正确性 + 代码质量）

对 `--target plan/ui/global`：输入材料按上表"输入材料"列收集，不依赖 git diff。

## 执行

### Step 1: 收集审查材料

**target=code（默认）**：
- 获取代码变更：`git diff`（未提交）或 `git diff HEAD~1`（最近一次提交）
- 读取 acceptance_criteria（如果有）
- 读取 `*-rules.md`（如果存在，作为约束参照）

**target=plan**：
- 读取 `docs/plans/*-plan.md` 全文
- 读取 `docs/plans/*-ideation.md`（需求源）、`*-design.md`（架构）、`*-ui.md` + `*-api.md`（细节）
- 不读 git diff（此时还没有代码）

**target=ui**：
- 获取 UI 代码变更：`git diff` 筛选 `*.tsx/*.vue/*.svelte/*.css` 等前端文件
- 读取 `*-ui.md` 中该 task 对应的页面章节（布局、组件、状态、视觉规范）
- **运行时证据**（强制）：启动 dev server，用 chrome-devtools 或 playwright MCP 做：
  - 截图（默认视口 + 移动视口）
  - DOM 快照（用于读 aria、语义化标签）
  - 触发每个状态（loading/空/错误）并截图
- 无法启动 dev server 时：在审查材料中标注"⚠️ 运行时证据缺失"，reviewer 对"视觉层级"维度降权

**target=global**：
- `git log main..HEAD --oneline`（所有交付 commit）+ `git diff main...HEAD --stat`（变更范围）
- 全部设计文档（ideation/design/ui/api/plan/rules/index）
- `docs/pipeline/state.yaml`（所有阶段完成状态）
- `autodev-verify` 产出的验收报告

### Step 1.5: API 新鲜度预检（强制）

<IMPORTANT>
在 spawn reviewer 之前，先对代码变更中涉及的外部库/API 做新鲜度检查：

1. 从 git diff 中提取 import 的外部库名称
2. 对每个关键库，调用 `context7` 查询最新文档：
   - 先用 `resolve-library-id` 解析库名
   - 再用 `query-docs` 查询当前代码使用的 API 是否仍然有效
3. 重点检查：
   - 函数签名是否变了（参数名、参数顺序、返回类型）
   - API 是否被标记为 deprecated
   - 是否有更推荐的替代 API
   - 版本号是否是最新稳定版

如果发现过时用法 → 在传给 reviewer 的材料中标注"⚠️ API 新鲜度问题"，reviewer 会在评分中反映。
如果 context7 不可用 → 用 WebSearch 搜 "{库名} latest API documentation {year}" 作为 fallback。
</IMPORTANT>

### Step 2: Spawn 独立 Reviewer Agent

spawn 一个独立的 reviewer agent（不同 context window），prompt 如下：

```
你是严格的代码审查者。你没有参与这段代码的编写。
你的默认倾向是**怀疑**——除非证据充分，否则不给 PASS。
LLM 生成的代码常见问题：看起来正确但边界情况未处理、
API 调用参数看似合理但实际不符合最新文档、测试只覆盖了 happy path。

请审查以下代码变更：

{git diff}

验收标准：
{acceptance_criteria，如果有}

编码约束：
{rules.md 内容，如果有}

API 新鲜度预检结果：
{Step 1.5 的检查结果}

按以下 4 个维度打分（1-10）—— **按 target 选择维度表**：

---

**若 target=code**（默认）：

1. **功能完整性**：需求/criteria 每一条是否满足？
   10 = 全部满足且边界情况处理完整
   5 = 核心功能满足但有遗漏
   1 = 关键需求未满足

2. **健壮性**：错误处理、类型安全、边界值
   10 = null/空/极值/网络错误全部处理
   5 = 只处理了 happy path
   1 = 有明显的崩溃风险

3. **设计一致性**：与 rules.md / 项目约定的一致程度
   10 = 完全符合设计约束
   5 = 大体一致但有偏离
   1 = 技术方案被替换或降阶

4. **代码质量**：可读性、命名、无冗余、无占位符
   10 = 清晰简洁，无 TODO/mock/空函数
   5 = 可读但有冗余或命名不一致
   1 = 有占位符代码或严重的代码异味

---

**若 target=plan**（代码开发前的方案审查）：

1. **需求覆盖度**：plan 的 task 是否覆盖 ideation/design 中所有 R 能力和 MVP 用户故事？
   10 = 每个 R 能力和用户故事至少有一个 task，且能力-task 映射显式
   5 = 大部分覆盖但有 1-2 项漏掉或只隐式覆盖
   1 = 有核心能力完全未被规划

2. **技术合理性**：技术选型是否符合 design.md，是否有过度工程 / 降阶 / 自造轮子？
   10 = 选型符合设计约束，**优先复用了 oss-scan 清单中的候选方案**，无降阶信号词（for now/simplified/TODO/mock）
   5 = 选型合理但有部分重复造轮子或未充分复用开源方案
   1 = 技术方案与 design.md 冲突，或大量自写应用开源替代的东西

3. **任务可执行性**：每个 task 是否独立可实现、acceptance_criteria 清晰、粒度恰当？
   10 = 每个 task 有明确的 acceptance_criteria、输入输出、依赖关系，粒度适合单个 subagent 一次完成
   5 = 部分 task 粒度过大或 acceptance 模糊
   1 = 存在无法独立执行的 task

4. **风险与依赖识别**：关键外部依赖、数据迁移、版本兼容、性能瓶颈是否被识别并有应对？
   10 = 所有高风险点（外部 API 配额、DB 迁移顺序、破坏性变更）都有显式风险项和应对
   5 = 识别了主要风险但缺应对措施
   1 = 忽略关键风险（如无数据迁移计划、无回滚路径）

---

**若 target=ui**（UI 代码实现后的界面与交互审查）：

1. **视觉层级与信息密度**：重要信息是否突出、是否存在信息过载、间距/对齐/对比度是否符合视觉规范？
   10 = 清晰的视觉层级，主操作/主信息一眼可见，次要信息折叠或降色
   5 = 基本可用但层级弱，或密度偏高/偏低
   1 = 视觉混乱、重点不突出、难以扫读

2. **交互流畅度**：用户完成核心任务的操作路径是否最短、反馈是否及时、是否遵循"能内联就不跳转"原则？
   10 = 核心任务路径≤3 步，关键操作有即时视觉反馈（hover/active/loading）
   5 = 路径可完成但有不必要的跳转或确认步骤
   1 = 操作路径长、反馈缺失、存在死胡同

3. **状态完整性**：ui.md 中该页面的 4 种状态（loading / empty / error / success）是否全部实现且有差异化设计？
   10 = 4 种状态均有实现，empty 有引导、error 有重试、loading 有骨架/占位
   5 = 3 种状态实现，但 empty 或 error 只是文字提示
   1 = 只实现 happy path，缺 2+ 种状态

4. **与 ui.md 一致性**：布局区域、组件清单、视觉规范（配色/字体/圆角/阴影/**图标系统**）、动效规范是否与 ui.md 对齐？
   10 = 完全符合 ui.md 指定的布局/组件/视觉/动效/图标，**无 emoji 作为 UI 图标**
   5 = 布局和组件一致，但视觉/动效有偏离，或有少量 emoji 混用
   1 = 布局或组件与 ui.md 不符，**或大量使用 emoji 替代 icon 库**

**emoji 扫描（强制 FAIL 条件）**：此维度有零容忍子检查。reviewer 在评分前必须对 UI 代码 diff 执行：
- 正则扫描 `*.tsx/*.jsx/*.vue/*.svelte` 中 JSX text 节点和 template 文本中的 emoji unicode 范围（`[\u{1F300}-\u{1FAFF}]`、`[\u{2600}-\u{27BF}]`、`[\u{1F000}-\u{1F9FF}]`）
- 排除白名单：i18n 文案 JSON 文件、代码注释（`//`、`/* */`、`<!-- -->`）、UGC 数据展示的字符串变量、README
- 任何命中 → 此维度直接判 ≤ 3 分（FAIL），在 SUGGESTIONS 中列出具体文件、行号、应替换为的 icon 组件名

**图标库合规子检查**：
- 验证 `package.json`（或等价文件）已引入 ui.md 第 6 项"图标系统"指定的 icon 库
- 验证 UI 组件中实际使用该库（grep import 语句）
- 未引入或未使用 → 此维度 ≤ 4 分

运行时证据（截图 + DOM 快照）是 ui 维度评分的必备依据，缺证据时该维度最高 5 分。

---

**若 target=global**（全部完成后的全局审查）：

1. **需求对齐**：ideation 中的 MVP 功能、用户故事、核心价值链是否在最终实现中全部落地？
   10 = 每个 MVP 功能和用户故事都有对应的代码入口和端到端路径，无遗漏
   5 = 核心功能全部落地，1-2 个次要功能缺失或简化
   1 = 存在 MVP 级功能未实现

2. **架构一致**：实现是否符合 design.md 的组件划分、能力-组件映射、数据流？
   10 = 实际代码结构与 design.md 的组件/模块/数据流一一对应
   5 = 大体一致但有个别组件职责偏移
   1 = 出现与 design.md 冲突的模块（如设计说用服务 A，实际用了 B）

3. **端到端可用**：核心用户故事能否从入口开始到完成走通？是否存在断链（页面跳转 404、API 未接、状态丢失）？
   10 = 核心 happy path 全部可走通，关键异常路径也有兜底
   5 = happy path 可走通但有异常路径缺失
   1 = 存在核心路径断链

4. **文档-代码对齐**：rules.md 中的编码约束、ui.md 的视觉规范、api.md 的契约是否在代码中被遵守？
   10 = 所有文档约束在代码中有对应实现或 lint 规则保障
   5 = 主要约束遵守，个别视觉/文案规范未落地
   1 = 多处与文档约束冲突（如 rules.md 禁用某库但代码引入了）

输出格式（严格遵循）：

SCORES:
- 功能完整性: {N}/10
- 健壮性: {N}/10
- 设计一致性: {N}/10
- 代码质量: {N}/10

API_FRESHNESS:
- {库名}: ✅ 最新 / ⚠️ 有过时用法 — {具体说明}

VERDICT: PASS / NEEDS_IMPROVEMENT / FAIL
（PASS = 4 个维度都 ≥ 7 且无 API 过时问题；FAIL = 任何维度 < 5 或有严重 API 过时；其余 = NEEDS_IMPROVEMENT）

DETAILS:
- [检查项 1]: ✅ / ❌ — {具体原因}
- [检查项 2]: ...

SUGGESTIONS: {具体修复/改进建议，包含文件名和行号}
```

### Step 3: 对抗迭代循环

```
Generator（实现者）生成代码
  ↓
Evaluator（reviewer）打分
  ↓
├── PASS（4 个维度都 ≥ 7 且 API 无过时）→ 完成，输出审查报告
├── NEEDS_IMPROVEMENT → 检查分数趋势：
│   ├── 分数比上轮高 → Generator 在当前方向继续改进
│   └── 分数比上轮低或持平（连续 2 轮）→ pivot 换方案
└── FAIL（任何维度 < 5 或 API 严重过时）→ Generator 修复 → reviewer 再审
```

### Step 4: Pivot 机制

当 NEEDS_IMPROVEMENT 且连续 2 轮分数无提升时：
1. Generator 不再在现有代码上修补
2. 重新读取需求/acceptance_criteria
3. 考虑完全不同的实现方案
4. 从头重写（git stash 保存旧版本）

### Step 5: 迭代上限

- 最多 **5 轮** review
- 5 轮后仍未 PASS → 输出最终分数和未解决问题列表
- 如果被编排器调用 → 记录 `review_result` 到 plan.md，继续下一个 task
- 如果独立使用 → 展示报告给用户

## 维度变体（仅对 target=code 生效）

target=code 时，维度 3 "设计一致性" 可替换：

| 调用方 | 维度 3 |
|-------|--------|
| autodev Phase 6 | 设计一致性（与 design.md 的一致程度）|
| autodev-add Step 8 | 集成安全（与现有代码的兼容性）|
| autodev-iterate Step 5 | 最小改动（是否只改了必要的）|
| 独立使用 | 设计一致性（默认）|

调用时可通过参数指定：`/autodev-review --dimension3 "集成安全"`

target=plan/ui/global 时，4 维度固定，不接受 `--dimension3` 参数。

## 失败回退路径

当 VERDICT = FAIL 或 5 轮后未 PASS，按 target 回退到对应阶段：

| target | FAIL 回退到 | 记录位置 |
|--------|-----------|---------|
| code | 当前 task 继续修复；5 轮未过 → 记录到 plan.md 的 `review_result` 字段，继续下一 task | plan.md |
| plan | 回 Phase 5（`autodev-plan`），按 reviewer SUGGESTIONS 修正 plan.md；修正后重跑 plan-review | plan.md 顶部 `plan_review_history` 章节 |
| ui | 回 Phase 6 的该 UI task，按 SUGGESTIONS 修正 UI 代码；修正后重跑 ui-review | plan.md 该 task 的 `ui_review_result` 字段 |
| global | 按失败维度定位阶段：维度 1 → Phase 1/2（补需求）；维度 2 → Phase 2（改架构）；维度 3 → Phase 6（修代码）；维度 4 → Phase 5.5（补 rules）或 Phase 6（修代码）；修正后重跑 global-review | `docs/pipeline/global-review.md` |
