---
name: autodev-verify
description: 统一验收：契约验收 + 红线扫描 + 静态检查 + 运行时验证 + acceptance-testing。可独立使用，也被 autodev/autodev-add/autodev-iterate 自动调用。
metadata:
  promptSignals:
    phrases:
      - "autodev-verify"
      - "验收"
      - "验证功能"
      - "测试验收"
      - "检查完成度"
---

# AutoDev Verify — 统一验收

整合所有验证逻辑到一个 skill，从契约验收到运行时测试，分层执行。

## 使用场景

- **被编排器调用**：autodev Phase 7 / autodev-add Step 9 / autodev-iterate Step 6 自动调用
- **独立使用**：`/autodev-verify` 随时对当前项目做完整验收

## 验证分 5 层，按顺序执行

### Layer 1: 契约验收（对照 acceptance_criteria）

如果 `*-plan.md` 存在：
1. 读取所有 task，检查 `status` 字段
   - 有 `pending` 的 task → 报告未完成项
2. 对每个 `completed` task，检查 `acceptance_criteria` 逐条是否满足
3. 检查 `decision` 字段 — 偏离设计的决策是否合理
4. 检查 `review_result` 字段 — GAN 审查分数是否达标

如果 `*-plan.md` 不存在：跳过此层，进入 Layer 2。

### Layer 2: 红线扫描

参考 `autodev-shared/checklists/quality-redlines.md`，搜索项目代码（排除 node_modules、.venv、dist、build 等）：

**扫描 1：占位代码（红线 1）**
- `TODO`、`FIXME`、`HACK`、`XXX`、`stub`
- `pass`（Python）/ 空函数体（JS/TS）
- `NotImplementedError`
- 硬编码空集合作为核心数据源

**扫描 2：Mock 数据（红线 2）**
- 变量名含 `mock`/`dummy`/`fake`/`placeholder`/`sample`
- 对照 `*-api.md`（如果有）检查端点是否有真实实现

**扫描 3：降阶实现（红线 3）**
- 对照 `*-design.md`（如果有）检查技术选型是否一致

**扫描 4：版本一致性（红线 4）**
- 对照设计文档技术选型表，检查 package.json/pyproject.toml 的依赖版本

**处理：** 发现问题 → 列出具体位置和红线编号。如果被编排器调用 → 回退到实现阶段修复。

### Layer 3: 静态检查

根据项目类型执行：

| 检测到 | 执行 |
|-------|------|
| tsconfig.json | `npx tsc --noEmit` |
| pyproject.toml | `ruff check .` 或 `pytest` |
| go.mod | `go vet ./...` |
| Cargo.toml | `cargo check` |

如果有测试框架：
| 检测到 | 执行 |
|-------|------|
| vitest.config* | `npx vitest run` |
| jest.config* | `npx jest` |
| pytest.ini / pyproject.toml [tool.pytest] | `pytest` |

### Layer 4: 运行时验证

**有 API 时**：
- 读取 `*-api.md`（如果有）获取端点清单
- 对关键端点构造请求，验证返回格式和状态码
- 不只是检查 200 OK，要检查响应中的关键字段

**有前端时**：
- 读取 `*-ui.md`（如果有）获取页面清单和用户流程
- 如果有 Playwright MCP → 自动化 UI 测试
- 如果没有 → 描述操作步骤让用户手动验证

**有旅程连续性需求时**（涉及多步流程的改动）：
- 上游衔接：从上一步传来的数据/状态是否正确
- 下游传递：这一步的输出下一步是否能消费
- 旁路影响：分支路径是否都正常

### Layer 5: acceptance-testing（完整功能测试）

调用 `acceptance-testing` skill：
- 传入 `*-ui.md`（页面清单）和 `*-api.md`（端点清单）
- 传入 `*-plan.md`（acceptance_criteria）
- 执行端到端功能验证

**注意**：如果是独立调用且只想做快速检查，可以只跑 Layer 1-3，跳过 Layer 4-5。
用法：`/autodev-verify --quick`（只跑契约+红线+静态）

## 输出

验收报告：
```
## 验收报告

### Layer 1: 契约验收
- Task 1: ✅ 全部 criteria 满足
- Task 2: ✅ 全部 criteria 满足
- Task 3: ⚠️ criteria 3 未满足（响应时间 > 500ms）

### Layer 2: 红线扫描
- 红线 1（占位）: ✅ 无违规
- 红线 2（Mock）: ✅ 无违规
- 红线 3（降阶）: ✅ 一致
- 红线 4（版本）: ⚠️ lodash 版本不一致（design: 4.17.21, actual: 4.17.15）

### Layer 3: 静态检查
- tsc: ✅ 无类型错误
- vitest: ✅ 47/47 通过

### Layer 4: 运行时验证
- GET /api/search: ✅ 200, 返回结果 > 5 条
- POST /api/export: ❌ 超时（> 10s）

### Layer 5: acceptance-testing
- 搜索流程: ✅
- 导出功能: ❌ 超时

### 总结
通过: 4/5 层
需修复: Task 3 响应时间 + 导出超时 + lodash 版本
```

## 被编排器调用时的行为

| 调用方 | 默认执行层 | 失败行为 |
|-------|----------|---------|
| autodev Phase 7 | 全部 5 层 | 回退 Phase 6 |
| autodev-add Step 9 | 全部 5 层 | 回退 Step 8 |
| autodev-iterate Step 6 | Layer 1-4（按变更类型选择性跳过）| 回退 Step 5 |
| 独立使用 | 全部 5 层 | 输出报告 |
| 独立使用 --quick | Layer 1-3 | 输出报告 |
