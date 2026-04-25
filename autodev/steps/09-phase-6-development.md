# 步骤 9：阶段 6 — 开发

**目标**：调用 `superpowers:subagent-driven-development` 执行编码，每个 task 完成后调用 `autodev-review` 做 GAN 式对抗审查。

## 执行

调用 `superpowers:subagent-driven-development` 技能，读取实施计划。
如果 `use_worktree: true` 且尚未在 worktree 中，先创建 worktree。

## 上下文加载策略（Token 效率优先）

每个 subagent 的任务描述中，必须包含以下精炼上下文：
1. `*-index.md` 全文（< 100 行，项目全景）
2. `*-rules.md` 全文（< 80 行，编码约束）
3. `*-plan.md` 中**当前任务**对应的章节（不是全文）

**按需加载**：仅当 subagent 需要某个模块的详细设计时，在任务描述中指示它读取对应文档：
- 需要页面布局详情 → 读 `*-ui.md` 对应页面章节
- 需要 API 接口详情 → 读 `*-api.md` 对应端点章节
- 需要架构决策理由 → 读 `*-design.md` 对应章节

**禁止**：不要把全部设计文档塞进 subagent 的初始 prompt。

## 质量红线传递（必须传给每个 subagent）

<IMPORTANT>
每个 subagent 的任务描述中，必须附加以下质量约束：

```
质量红线（违反任何一条则任务未完成）：
1. 禁止占位：不允许 TODO、空函数体、pass、NotImplementedError。功能在范围内就必须完整实现。
2. 禁止 Mock：不允许 mock/dummy/fake/placeholder 数据替代真实实现。需要调 API 就调 API。
3. 禁止降阶：按设计文档指定的技术方案实现，不得替换为"更简单的替代方案"。
4. 版本正确 + API 新鲜度：
   - 安装依赖时使用设计文档技术选型表中标注的版本号，不要凭记忆用旧版本。
   - 使用任何外部库的 API 之前，必须先用 context7 查询该库的最新文档：
     a. 调用 context7 的 resolve-library-id 解析库名
     b. 调用 context7 的 query-docs 查询你要用的 API 的最新用法
     c. 按查到的最新文档写代码，不要依赖训练数据中的旧语法
   - 如果 context7 不可用，用 WebSearch 搜 "{库名} API documentation {当前年份}" 作为 fallback
   - 绝对禁止凭记忆使用 API。你记忆中的 API 可能已经 deprecated 或改了签名。
```

不传递质量约束的 subagent 调度视为阶段未完成。
</IMPORTANT>

## 前端代码质量规则

当任务涉及前端页面或组件时（判断依据：`*-ui.md` 存在且该任务对应 ui.md 中的某个页面），必须：

1. **传入 ui.md 作为结构约束**：subagent 在实现前端页面时，必须读取 `*-ui.md` 中该页面的布局区域、组件清单、状态设计
2. **通过 `frontend-design` skill 生成 UI 代码**：所有用户可见的页面和组件，必须调用 `frontend-design` skill 来生成代码
3. **读取前端技术栈偏好**：检查 `config.yaml` 的 `phases.frontend` 配置（如有）
4. **强制使用 icon 库，禁止 emoji 作为 UI 图标**（红线 6）：
   - 读取 `*-ui.md` 视觉规范的"图标系统"章节，按其指定的 icon 库（Lucide / Heroicons / Phosphor / Radix Icons / Tabler / Remix Icon 等）引入依赖
   - 所有按钮、导航、状态指示、装饰元素使用 icon 库组件（如 `import { Trash2, X, Check } from 'lucide-react'`），尺寸和描边粗细遵循 ui.md 规范
   - **禁止**在 JSX / Vue template / Svelte template 的 text 节点中写 emoji（🗑️ ✅ ❌ ⚠️ 🔥 📭 等）
   - 若 ui.md 第 6 项"图标系统"未填写 → **停止**编码，先回 Phase 3 的 `autodev-ui` step 5b 补齐该决策，避免后续 UI GAN 必定失败
   - Emoji 仅允许出现在：i18n 文案 JSON 的值、代码注释、UGC 展示、README 等文档

**无前端的项目（CLI、纯 API）**：跳过以上规则。判断依据：`*-ui.md` 不存在或 `config.yaml` 中 `frontend.stack: "none"`。

- **输入**：`*-plan.md` + `*-ui.md`（前端任务时）
- **产出**：已提交的代码

## GAN 式对抗审查（双层：code + ui）

<IMPORTANT>
每个 task 完成后，必须**按顺序**跑以下对抗审查层：
1. **Code GAN**（所有 task 必跑）
2. **UI GAN**（仅 UI/前端 task 必跑）—— 这是 GAN 3 处的第 2 处：**UI 代码实现以后，对界面和交互进行对抗审查**

两层都 PASS 后，才能把当前 task 标记为 completed。
</IMPORTANT>

### Layer 1：Code GAN（所有 task）

调用 `autodev-review` skill 进行代码层对抗审查。

```
Skill(
  skill="autodev-review",
  args="--target code"
)
```

传入参数（会被 skill 自动读取）：
- 维度 3 = "设计一致性"（autodev 默认）
- acceptance_criteria 从 plan.md 当前 task 读取

`autodev-review --target code` 会自动执行：
- API 新鲜度预检（context7 验证）
- 4 维度打分（功能完整性 / 健壮性 / 设计一致性 / 代码质量）→ PASS/NEEDS_IMPROVEMENT/FAIL
- 最多 5 轮迭代 → pivot 机制

### Layer 2：UI GAN（仅 UI/前端 task）

**判断是否需要 UI GAN**：

当前 task 涉及前端页面或组件时必须跑。判断依据：
- `*-ui.md` 存在，且
- 当前 task 在 plan.md 中对应 ui.md 的某个页面/组件，或
- 本次 git diff 含 `*.tsx/*.vue/*.svelte/*.css/*.scss` 等前端文件

CLI / 纯后端 API / 数据处理 task → 跳过 UI GAN。

**调用方式**：

```
Skill(
  skill="autodev-review",
  args="--target ui"
)
```

`autodev-review --target ui` 会自动执行：
1. 启动 dev server（自动探测 package.json 的 `dev` / `start` 脚本）
2. 用 chrome-devtools 或 playwright MCP 导航到当前 task 对应的页面
3. 采集证据：
   - 截图（默认视口 1440×900 + 移动视口 375×812）
   - DOM snapshot（用于 reviewer 读语义标签/aria/层级）
   - 触发每个状态（loading / empty / error / success）并分别截图
4. Spawn 独立 UI reviewer agent，按 4 维度打分：
   - 视觉层级与信息密度
   - 交互流畅度
   - 状态完整性
   - 与 ui.md 一致性
5. 最多 3 轮迭代（UI 迭代代价高，比 code GAN 少 2 轮）

**运行时证据是必备项**：dev server 启动失败或 MCP 浏览器不可用时，记录 `ui_review_result: skipped_no_runtime_evidence` 并在 Phase 7 的 Global GAN 里补证。

### 两层审查完成后

两层都 PASS 后，更新 plan.md 当前 task：
- `status: pending` → `status: completed`
- `code_review_result: pass` / `ui_review_result: pass`（如适用）
- 偏离决策记录在 `decision:` 字段

任一层 FAIL 且 5 轮未过：
- 不得标记 task 为 completed
- 记录 `code_review_result: fail_accepted` 或 `ui_review_result: fail_accepted` 并在 plan.md 注明具体问题
- 睡眠模式下继续下一 task，交由 Phase 7 Global GAN 做最终裁决；非睡眠模式下询问用户

--- 步骤 9 完成 ---
