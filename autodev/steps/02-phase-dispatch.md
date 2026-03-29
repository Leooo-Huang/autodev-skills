# 步骤 2：阶段调度协议

**目标**：定义每个阶段执行的通用 PRE/RUN/POST/GATE 流程。

## PRE — 阶段前

1. 更新 `state.yaml`：
   - `current_phase = N`
   - `phases.N.status = "in_progress"`
   - `phases.N.started_at = "{当前 ISO 时间戳}"`
2. 写入 state.yaml 到磁盘
3. 在 TodoWrite 中将阶段 N 标记为 `in_progress`
4. 展示："开始阶段 {N}：{阶段名称}"

## RUN — 执行阶段

根据当前阶段编号，读取对应的 steps 文件获取详细执行指令：
- 阶段 1 → `steps/03-phase-1-ideation.md`
- 阶段 2 → `steps/04-phase-2-spec.md`
- 阶段 3 → `steps/05-phase-3-ui.md`
- 阶段 4 → `steps/06-phase-4-api.md`
- 阶段 5 → `steps/07-phase-5-planning.md`
- 阶段 5.5 → `steps/08-phase-5.5-index-rules.md`
- 阶段 6 → `steps/09-phase-6-development.md`
- 阶段 7 → `steps/10-phase-7-verification.md`
- 阶段 8 → `steps/11-phase-8-delivery.md`

## POST — 阶段后

1. 验证阶段输出文件存在（参考 `autodev-shared/helpers.md` 的 `#Verify-Phase-Output`）
2. 更新 `state.yaml`（参考 `autodev-shared/helpers.md` 的 `#Update-State-Yaml`）：
   - `phases.N.status = "completed"`
   - `phases.N.completed_at = "{当前 ISO 时间戳}"`
   - `phases.N.output = "{输出文件路径}"`
3. 写入 state.yaml
4. 在 TodoWrite 中将阶段 N 标记为 `completed`
5. 展示："阶段 {N} 完成。输出：{文件路径}"

## GATE — 门控检查

参考 `autodev-shared/helpers.md` 的 `#Gate-Check`。

```
读取 config.yaml 的 gates 字段
如果 gates[N] 存在 且 gates[N].requires == "approval":
    更新 state.yaml:
        status = "paused_at_gate"
        paused_at_gate = N
    展示:
        "阶段 {N}（{名称}）完成。
         输出：{文件路径}
         {gates[N].message}

         审查完毕后，调用 /autodev --resume 继续。"
    停止执行
否则:
    记录到 gate_history: { phase: N, auto_approved: true, at: "{时间戳}" }
    继续到阶段 N+1
```

<IMPORTANT>
门控在阶段完成后、下一阶段开始前触发。它控制的是进入下一阶段的权限，不是退出当前阶段的权限。
</IMPORTANT>

--- 步骤 2 完成 ---
