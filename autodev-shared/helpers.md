# AutoDev 共享操作片段

## #Read-Design-Docs
读取 `docs/plans/` 下的设计文档：
1. 如果存在 `*-index.md`，优先读取它获取全景信息
2. 如果存在 `*-rules.md`，读取编码规则
3. 按需读取具体文档：`*-ideation.md`、`*-design.md`、`*-ui.md`、`*-api.md`、`*-plan.md`

## #Update-State-Yaml
更新 `docs/pipeline/state.yaml`：
- 修改 `current_phase`、`phases.N.status`、`phases.N.started_at/completed_at`
- 每次修改后立即写入磁盘

## #Check-Stub-Code
桩代码扫描（5 种信号）：
| 信号 | 搜索模式 |
|------|---------|
| 硬编码空值 | `= []`, `= {}`, `= null`, `= ''` 作为核心数据源 |
| TODO/FIXME | `// TODO`, `// FIXME`, `// HACK`, `// stub` |
| Mock 数据 | 变量名含 `mock`, `dummy`, `fake`, `placeholder` |
| 未调用 API | 导入了函数但未调用 |
| 条件短路 | 核心逻辑被空数据跳过 |

## #Verify-Phase-Output
验证阶段输出文件：
1. 读取输出文件，确认非空
2. 检查关键章节存在（ideation→"MVP"节，design→"架构"节，ui→"页面清单"节，api→"API 端点"节）

## #Gate-Check
门控检查：
1. 读取 `config.yaml` 的 gates 字段
2. 如果 gates[N] 存在且 requires == "approval"：暂停并通知用户
3. 否则：自动通过，记录到 gate_history

## #Generate-Handoff
生成跨 session 交接文档（Harness Engineering）：
1. 读取 `docs/pipeline/state.yaml` 获取当前阶段和主题
2. 如果在 Phase 6，读取 `plan.md` 统计 task 完成情况（completed/pending/in_progress）
3. 从 plan.md 收集所有 `decision:` 字段
4. 按 `autodev-shared/templates/handoff.md` 模板写入 `.claude/handoff.md`
5. 确认写入成功

## #Update-Plan-Status
更新 plan.md 中指定 task 的状态（Harness Engineering — 活文档）：
1. 读取 plan.md
2. 找到目标 task 的 `status:` 行
3. 将 `status: pending` 替换为 `status: completed`
4. 如有偏离决策，在 task 下添加 `decision: {原因}`
5. 写入磁盘
