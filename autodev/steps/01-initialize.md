# 步骤 1：初始化协议

**目标**：解析用户输入，检查现有状态，创建配置和 state.yaml。

## 1.1 解析输入

从 `$ARGUMENTS` 中提取：
- **主题**：去掉所有标志后的文本部分
- **标志**：
  - `--resume`：从断点恢复
  - `--sleep`：激活睡眠模式（Ralph Loop + 清除所有门控）
  - `--skip-phase N`：跳过阶段 N
  - `--abort`：中止当前流水线

生成 `topic_slug`：主题小写，空格和特殊字符替换为连字符，最长 50 字符。

## 1.2 检查现有状态

读取 `docs/pipeline/state.yaml`：

| 状态 | 动作 |
|------|------|
| 文件不存在 | 全新开始 → 步骤 1.3 |
| `status: in_progress` | 恢复模式 → 读取 `steps/12-resume-protocol.md` |
| `status: paused_at_gate` | 用户正在审批 → 清除门控，继续下一阶段 |
| `status: completed` | 询问用户：新建流水线还是查看旧结果？ |
| `status: failed` | 显示失败信息，询问：重试还是中止？ |

## 1.3 初始化配置和状态

**如果 `docs/pipeline/config.yaml` 不存在**，创建默认配置：

```yaml
# AutoDev 流水线配置
# 首次运行时由 /autodev 自动创建。可编辑以自定义行为。

pipeline:
  mode: auto                    # auto = 所有阶段不停顿运行

# 门控：哪些阶段转换需要用户确认
# 只列出需要确认的阶段编号。未列出的 = 自动通过。
# 门控在该阶段完成后、下一阶段开始前触发。
gates: {}
  # 示例：产品创意后暂停，让用户审查功能清单
  # 1:
  #   requires: approval
  #   message: "请审查功能清单和 MVP 定义"
  # 2:
  #   requires: approval
  #   message: "请审查产品规格和架构设计"

# 睡眠模式：全自动，Ralph Loop 保持会话存活
sleep_mode: false

# Ralph Loop 设置（仅当 sleep_mode: true 时生效）
ralph_loop:
  max_iterations: 50
  completion_promise: "AUTODEV PIPELINE COMPLETE"

# 各阶段的特定配置
phases:
  ideation:
    max_iterations: 3           # 调研-创意循环最大迭代次数
  product_spec:
    # superpowers:brainstorming 的配置由该技能自身控制
  ui_ux:
    include_wireframes: true    # 是否生成 ASCII 线框图
  api:
    include_schemas: true       # 是否生成详细的请求/响应 schema
  planning:
    execution_style: auto       # auto | subagent | parallel
  development:
    use_worktree: true
    branch_pattern: "autodev/{topic-slug}"
  # 前端偏好（可选，影响阶段 2 技术选型和阶段 6 代码生成）
  # frontend:
  #   stack: "nextjs"              # nextjs | vite-react | nuxt | none
  #   ui_library: "shadcn"         # shadcn | mui | antd | custom
  #   css: "tailwind"              # tailwind | css-modules | styled-components
  delivery:
    default_action: ask         # ask | merge | pr | keep

# 部署配置
deployment:
  auto_deploy: true             # true = 代码集成后自动触发部署
                                # false = 跳过部署，仅做代码集成
  # 部署平台从 design.md 的「部署策略」章节读取，不在此处配置。
  # 如果 design.md 未指定平台，自动跳过部署。
```

**创建 `docs/pipeline/state.yaml`：**

```yaml
topic: "{用户输入的主题}"
topic_slug: "{生成的 slug}"
status: in_progress
current_phase: 1
paused_at_gate: null
started_at: "{ISO 时间戳}"
completed_at: null
phases:
  1: { name: ideation, status: pending, started_at: null, completed_at: null, output: null, error: null, retries: 0 }
  2: { name: product-spec, status: pending, started_at: null, completed_at: null, output: null, error: null, retries: 0 }
  3: { name: ui-ux, status: pending, started_at: null, completed_at: null, output: null, error: null, retries: 0 }
  4: { name: api, status: pending, started_at: null, completed_at: null, output: null, error: null, retries: 0 }
  5: { name: planning, status: pending, started_at: null, completed_at: null, output: null, error: null, retries: 0 }
  6: { name: development, status: pending, started_at: null, completed_at: null, output: null, error: null, retries: 0 }
  7: { name: verification, status: pending, started_at: null, completed_at: null, output: null, error: null, retries: 0 }
  8: { name: delivery, status: pending, started_at: null, completed_at: null, output: null, error: null, retries: 0 }
gate_history: []
```

**确保目录存在：** `docs/pipeline/` 和 `docs/plans/`。

## 1.4 创建任务追踪

用 TodoWrite 创建 8 个条目：
1. 阶段 1：产品创意
2. 阶段 2：产品规格
3. 阶段 3：UI/UX 设计
4. 阶段 4：API 设计
5. 阶段 5：规划
6. 阶段 6：开发
7. 阶段 7：验证
8. 阶段 8：交付

--- 步骤 1 完成 ---
