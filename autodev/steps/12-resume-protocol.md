# 步骤 12：恢复协议

**目标**：从断点恢复流水线执行。

当 `/autodev --resume` 或 `/autodev` 发现已有 `state.yaml` 时：

1. **读取 `state.yaml`**
2. **根据状态判断：**

| 发现的状态 | 恢复动作 |
|-----------|---------|
| `status: paused_at_gate`，`paused_at_gate: N` | 用户已审批。清除门控（`paused_at_gate = null`，`status = in_progress`），记录 gate_history，启动阶段 N+1 |
| `status: in_progress`，阶段 N `status: in_progress`，无 `completed_at` | 阶段 N 执行中途崩溃。重启阶段 N |
| `status: in_progress`，阶段 N `status: completed`，阶段 N+1 `status: pending` | 阶段间过渡时崩溃。启动阶段 N+1 |
| `status: failed`，阶段 N `status: failed`，`retries < 2` | 重试失败的阶段（retries + 1） |
| `status: failed`，阶段 N `status: failed`，`retries >= 2` | 询问用户：重试、跳过该阶段、还是中止？ |
| `status: completed` | "流水线已完成。是否开始新的流水线？" |
| state.yaml 不存在但 `docs/plans/` 下有输出文件 | 从输出文件重建状态 |

3. **展示恢复信息：** "恢复流水线：{主题}，从阶段 {N} 继续。"

## 部分输出恢复

如果阶段崩溃后输出文件已存在但可能不完整：
1. 读取输出文件
2. 检查关键章节是否存在（参考 `autodev-shared/helpers.md` 的 `#Verify-Phase-Output`）
3. 如果可用：标记阶段完成，继续
4. 如果不完整：删除文件，重新执行该阶段

--- 步骤 12 完成 ---
