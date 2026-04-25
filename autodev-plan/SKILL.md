---
name: autodev-plan
description: 生成带契约式验收标准的实施计划（acceptance_criteria + status），并执行降阶信号词扫描。可独立使用，也被 autodev/autodev-add 自动调用。
metadata:
  promptSignals:
    phrases:
      - "autodev-plan"
      - "生成计划"
      - "实施计划"
      - "开发计划"
---

# AutoDev Plan — 契约式实施计划

生成结构化的实施计划，每个 task 带有可验证的 acceptance_criteria。

## 使用场景

- **被编排器调用**：autodev Phase 5 / autodev-add Step 6 自动调用
- **独立使用**：`/autodev-plan` 为任何需求生成实施计划

## 输入

自动检测（按优先级）：
1. 如果提供了具体需求描述 → 用它
2. 如果 `*-ideation.md` + `*-design.md` + `*-ui.md` + `*-api.md` 存在 → 读取全部作为上下文
3. 如果只有部分文档存在 → 读取已有的

## 执行

### Step 1: 调用 writing-plans

调用 `superpowers:writing-plans` 生成实施计划。

自动模式下，当 writing-plans 提示选择执行方式时，自动选择 "subagent-driven"。

### Step 2: 补充 acceptance_criteria

检查生成的 plan 中每个 task 是否包含 `acceptance_criteria` 和 `status` 字段。
如果缺失，为每个 task 补充：

```
### Task N: {名称}
acceptance_criteria:
  - {可检查的事实 1}（如：文件 X 存在且导出函数 Y）
  - {可检查的事实 2}（如：运行 `npm test` 通过）
  - {可检查的事实 3}（如：API 端点返回 200 且 body 符合 schema）
status: pending
```

**禁止模糊标准：**
- ❌ "代码质量好"、"性能可接受"、"用户体验流畅"
- ✅ "响应时间 < 500ms"、"测试覆盖率 > 80%"、"无 TypeScript 类型错误"

### Step 3: 降阶信号词扫描（红线 3）

搜索以下信号词，发现任何一个都必须修正：
- `for now`、`later`、`暂时`、`先用`、`简单起见`、`as a workaround`
- `placeholder`、`mock`、`stub`、`dummy`、`fake`
- `先...后面再...`、`to be replaced`、`will integrate later`
- `简化版`、`简易版`、`lightweight version`

发现信号词时：
1. 定位该步骤对应的设计文档原文
2. 按设计文档方案重写该步骤
3. 如果设计文档也模糊：WebSearch 查最佳实践

### Step 4: 版本标注（红线 4）

计划中涉及的每个库/工具安装步骤，必须标注版本号（来自设计文档的技术选型表）。

## 产出

`docs/plans/YYYY-MM-DD-{slug}-plan.md`（或追加到已有 plan.md）

## 增量模式（被 autodev-add 调用时）

不创建新 plan 文件，而是在已有 `*-plan.md` 中追加新功能的任务章节。
需额外考虑：
- 识别集成点（新功能需要修改哪些现有文件）
- 风险评估（哪些改动可能影响现有功能）
