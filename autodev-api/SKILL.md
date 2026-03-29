---
name: autodev-api
description: "API 设计：从产品规格和 UI 设计出发，定义数据模型、端点、请求响应格式和业务规则。当用户说 api设计、接口设计、数据模型 时触发。"
user-invocable: true
allowed-tools: [Read, Write, WebSearch, TodoWrite, AskUserQuestion]
---

# AutoDev API — 接口与数据设计

从产品规格和 UI/UX 设计出发，定义系统的数据模型和 API 接口。产出的是**设计文档**，不是代码。
核心原则：**从 UI 需求倒推。** 每个页面需要什么数据 -> 需要什么端点。

## 何时使用

- AutoDev 流水线的第 4 阶段（由编排器调度）
- 独立使用：已有产品规格和 UI 设计，需要定义 API
- 用户说"帮我设计 API"、"接口怎么定义"、"数据模型设计"

何时不使用：
- 需要写后端代码 -> 那是开发阶段
- 还没有 UI 设计 -> 先 `/autodev-ui`
- 还没有产品规格 -> 先 `superpowers:brainstorming`

## 步骤索引

| 步骤 | 文件 | 说明 |
|------|------|------|
| 1 | `steps/01-load-context.md` | 加载前序文档，获取 UI 数据需求汇总 |
| 2 | `steps/02-data-model.md` | 从 UI 倒推实体和数据模型 |
| 3 | `steps/03-endpoint-inventory.md` | 逐页扫描，列出所有 API 端点 |
| 4 | `steps/04-endpoint-detail.md` | 每个端点的请求/响应/错误/业务规则 |
| 5 | `steps/05-auth.md` | 认证方式和权限矩阵 |
| 6 | `steps/06-third-party.md` | 第三方 API 集成（如有） |
| 7 | `steps/07-api-conventions.md` | 统一的 API 风格约定 |
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

- **自动模式**：全程自动运行，从 UI 文档的数据需求倒推所有端点
- **门控模式**：步骤 2 后确认数据模型，步骤 5 后确认认证方案
- 流水线中建议自动模式，独立使用时建议在数据模型步骤后开门控

## 反模式

| 禁止 | 应该 |
|------|------|
| 凭空设计端点，不看 UI 需求 | 从 UI 页面的数据需求逐一倒推 |
| 端点没有对应的 UI 操作 | 每个端点标注「对应 UI」 |
| 只定义成功响应 | 每个端点定义完整的错误响应 |
| 业务规则写得模糊 | 具体规则（"1-100 字符，不含特殊字符"） |
| 预设将来可能需要的字段 | 只定义 UI 当前用到的字段 |
| 每个端点重复写分页/错误格式 | 统一约定放在 API 约定节 |
| 写后端代码或数据库 SQL | 只产出设计文档 |
| 忽略认证和权限设计 | 必须定义认证方式和权限矩阵 |
