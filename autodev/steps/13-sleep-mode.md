# 步骤 13：睡眠模式（Ralph Loop）

**目标**：全自动无人值守运行，通过 Ralph Loop 保持会话存活。

## 激活条件
使用 `--sleep` 标志或 `config.yaml` 中 `sleep_mode: true`。

## 激活流程

1. **覆盖所有门控**：临时将所有 gates 视为自动通过（不修改 config 文件，仅本次运行生效）
2. **对阶段 8**：若 `delivery.default_action == "ask"`，自动改为 `"pr"`
3. **调用 Ralph Loop**：

```
/ralph-loop 继续 autodev 流水线。
读取 docs/pipeline/state.yaml 获取当前进度。
如果全部 8 个阶段的 status 都是 completed，输出 completion promise。
否则调用 /autodev --resume 从断点继续。
--completion-promise 'AUTODEV PIPELINE COMPLETE'
--max-iterations {config.ralph_loop.max_iterations}
```

4. **更新 state.yaml**：`ralph_loop.active = true`，`ralph_loop.started_at = "{时间戳}"`

## 上下文压缩生存

Ralph Loop 能在上下文压缩后生存，因为：
- 状态存在 `docs/pipeline/state.yaml`（文件，不在内存中）
- 每次迭代都是全新的 "读状态 → 判断下一步 → 执行" 循环
- 不依赖任何内存中的状态

## Completion Promise

<IMPORTANT>
只有当 state.yaml 显示全部 8 个阶段的 status 都是 "completed" 时，才输出：
<promise>AUTODEV PIPELINE COMPLETE</promise>

绝不提前输出。读取 state.yaml 验证后再决定。
如果被卡住了、任务看起来不可能完成，也不要输出虚假的 promise。
</IMPORTANT>

--- 步骤 13 完成 ---
