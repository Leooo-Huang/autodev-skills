# 步骤 7：阶段 5 — 规划

**目标**：调用 `superpowers:writing-plans` 生成实施计划，并执行降阶信号词扫描。

## 执行

调用 `superpowers:writing-plans` 技能。

**衔接：** 读取全部 4 个设计文档（ideation、design、ui、api），作为规划的完整上下文传递。自动模式下，当 writing-plans 提示选择执行方式时，自动选择 "subagent-driven"。

## 计划质量扫描（红线 3+4 检查点）

<IMPORTANT>
writing-plans 产出计划后、标记阶段 5 完成之前，必须扫描计划文本：

**降阶信号词扫描（红线 3）**：搜索以下信号词，发现任何一个都必须修正：
- `for now`、`later`、`暂时`、`先用`、`简单起见`、`as a workaround`
- `placeholder`、`mock`、`stub`、`dummy`、`fake`
- `先...后面再...`、`to be replaced`、`will integrate later`
- `简化版`、`简易版`、`lightweight version`

发现信号词时的处理：
1. 定位该步骤对应的设计文档原文
2. 检查设计文档是否指定了具体方案
3. 如果是：按设计文档方案重写该步骤
4. 如果设计文档也模糊：WebSearch 查找最佳实践，选择效果最好的方案（不是最快的）

**依赖版本标注（红线 4）**：计划中涉及的每个具体库/工具安装步骤，必须标注版本号（来自 Phase 2 设计文档的技术选型表）。
</IMPORTANT>

- **输入**：`*-ideation.md` + `*-design.md` + `*-ui.md` + `*-api.md`
- **产出**：`docs/plans/YYYY-MM-DD-{slug}-plan.md`

> **注意**：Phase 5 完成后，执行 Phase 5.5 生成 index.md 和 rules.md，然后再进入 Phase 6。Phase 5.5 是 Phase 5 的 POST 步骤的一部分，不需要独立的 state.yaml 条目。

--- 步骤 7 完成 ---
