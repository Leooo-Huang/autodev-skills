# 步骤 14：错误处理

**目标**：定义阶段失败、跳过、中止的处理逻辑。

## 阶段失败

```
当阶段 N 执行失败:
1. 更新 state.yaml:
   - phases.N.status = "failed"
   - phases.N.error = "{错误信息，截断到 500 字符}"
   - phases.N.retries += 1

2. 如果 retries <= 2:
   展示: "阶段 {N} 失败：{错误}。正在重试（第 {retries}/2 次）..."
   重新执行 PRE → RUN 流程

3. 如果 retries > 2:
   更新 state.yaml: status = "failed"
   展示:
     "阶段 {N}（{名称}）在 2 次重试后仍失败。
      错误：{错误}

      选项：
      1. 修复问题后调用 /autodev --resume 重试
      2. 跳过此阶段：/autodev --skip-phase {N}
      3. 中止流水线：/autodev --abort"
   停止执行
```

## 跳过阶段

`/autodev --skip-phase N`：
1. 更新 `state.yaml`：`phases.N.status = "skipped"`
2. 继续到阶段 N+1
3. 注意：跳过前序设计阶段意味着后续阶段缺少上下文

## 中止流水线

`/autodev --abort`：
1. 更新 `state.yaml`：`status = "aborted"`，`completed_at = "{时间戳}"`
2. 如果 Ralph Loop 活跃：不输出 completion promise
3. 展示："流水线已中止。状态保留在 docs/pipeline/state.yaml。"

--- 步骤 14 完成 ---
