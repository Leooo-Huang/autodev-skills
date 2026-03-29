---
name: autodev-ui
description: "UI/UX 设计：从产品规格出发，设计信息架构、用户流程、页面结构和交互细节。当用户说 ui设计、交互设计、页面设计 时触发。"
user-invocable: true
allowed-tools: [Read, Write, WebSearch, TodoWrite, AskUserQuestion]
---

# AutoDev UI/UX — 界面与交互设计

从产品规格和功能清单出发，设计用户如何与产品交互。产出的是**设计文档**，不是代码。
核心原则：**从用户流程出发，不从技术组件出发。**

## 何时使用

- AutoDev 流水线的第 3 阶段（由编排器调度）
- 独立使用：已有产品规格/设计文档，需要做 UI/UX 设计
- 用户说"帮我设计界面"、"页面怎么布局"、"交互设计"

何时不使用：
- 需要写前端代码 -> `frontend-design`
- 还没想清楚做什么功能 -> 先 `/autodev-ideation`
- 还没有架构设计 -> 先 `superpowers:brainstorming`

## 步骤索引

| 步骤 | 文件 | 说明 |
|------|------|------|
| 1 | `steps/01-load-context.md` | 加载前序文档，提取 MVP 功能列表 |
| 2 | `steps/02-information-architecture.md` | 设计页面层级和导航关系 |
| 3 | `steps/03-user-flows.md` | 画出每个功能的用户操作流程 |
| 4 | `steps/04-page-design.md` | 每个页面的布局、组件、数据需求 |
| 5 | `steps/05-state-design.md` | 每个页面的所有状态（加载/空/正常/错误） |
| 5b | `steps/05b-visual-spec.md` | 视觉规范（配色、字体、间距、圆角、阴影） |
| 5c | `steps/05c-motion-spec.md` | 动效规范（动效程度、过渡参数、场景方案） |
| 6 | `steps/06-wireframes.md` | 关键页面的 ASCII 线框图 |
| 7 | `steps/07-responsive-strategy.md` | 移动端 vs 桌面端适配方案 |
| 8 | `steps/08-self-check.md` | 完整性自检（清单在 `checklists/`） |
| 9 | `steps/09-write-document.md` | 写入文档（模板在 `templates/`） |
| 10 | `steps/10-handoff.md` | 交接给下一阶段 |

## 执行规则

<IMPORTANT>
按步骤顺序执行。执行步骤 N 时，用 Read 工具读取对应的 steps/ 文件获取详细指令。
只加载当前步骤的文件，不要一次性读取所有步骤——这是为了节省 token 并保持专注。
每步完成后输出 `--- 步骤 N 完成 ---`。
共享资源在 `autodev-shared/` 目录下，按需读取。
</IMPORTANT>

## 交互模式

- **自动模式**：全程自动运行，适合流水线中的自动推进
- **门控模式**：步骤 2 后确认信息架构，步骤 4 后确认页面设计
- 流水线中建议自动模式，独立使用时建议开门控

## 反模式

| 禁止 | 应该 |
|------|------|
| 从技术组件出发设计页面 | 从用户任务出发 |
| 忽略空状态和错误状态 | 每个页面设计 4 种状态 |
| 所有信息默认展示 | 渐进式披露 |
| 为每个功能创建新页面 | 能内联展示的不跳转 |
| 写 CSS/HTML/组件代码 | 只产出设计文档 |
| 设计 MVP 以外的页面 | 严格限定 MVP 范围 |
| 数据需求写得模糊 | 明确列出每个页面的数据字段 |
