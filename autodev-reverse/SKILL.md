---
name: autodev-reverse
description: "逆向文档生成：读取现有代码库，生成 AutoDev 流水线格式的设计文档。当用户说 逆向文档、反向生成、从代码生成文档、reverse 时触发。"
user-invocable: true
allowed-tools: [Bash, Read, Write, Glob, Grep, TodoWrite, AskUserQuestion]
---

# AutoDev Reverse — 逆向文档生成

读取现有代码库，生成 AutoDev 流水线格式的设计文档。让历史项目、没有文档的项目、文档混乱的项目，都能接入 AutoDev 流水线进行后续开发。核心原则：**从代码中还原设计意图，不是翻译代码。**

## 何时使用

- 历史项目没有文档，需要生成
- 项目文档混乱、过时，需要重新整理
- 想让现有项目接入 AutoDev 流水线做后续迭代

何时不使用：
- 从零开始的新项目 → `/autodev` 或 `/autodev-ideation`
- 文档存在且准确，只需要更新进度 → `/autodev-sync`

## 步骤索引

| 步骤 | 文件 | 说明 |
|------|------|------|
| 1 | `steps/01-scan-codebase.md` | 扫描代码库（元数据、目录、Git、统计） |
| 2 | `steps/02-identify-architecture.md` | 识别架构（整体模式、技术选型、数据流） |
| 3 | `steps/03-extract-features.md` | 提取功能（路由、组件、数据模型、Git） |
| 4 | `steps/04-restore-ui.md` | 还原 UI 设计（页面、信息架构、用户流程） |
| 5 | `steps/05-restore-api.md` | 还原 API 设计（数据模型、端点、认证） |
| 6 | `steps/06-generate-docs.md` | 生成 AutoDev 格式文档 |
| 7 | `steps/07-verify-completeness.md` | 验证完整性 |
| 8 | `steps/08-generate-report.md` | 生成逆向报告 |

**辅助资源：**
- `templates/ideation.md` — ideation 文档模板
- `templates/design.md` — design 文档模板
- `templates/reverse-report.md` — 逆向报告模板
- `checklists/completeness.md` — 完整性验证清单
- `checklists/file-safety.md` — 文件删除安全规则

<IMPORTANT>
按步骤顺序执行。执行步骤 N 时，用 Read 工具读取对应的 steps/ 文件获取详细指令。
只加载当前步骤的文件，不要一次性读取所有步骤——这是为了节省 token 并保持专注。
每步完成后输出 `--- 步骤 N 完成 ---`。
共享资源在 `autodev-shared/` 目录下，按需读取。
</IMPORTANT>

## 大型代码库策略

对于大型项目（>500 文件），采用分层扫描：
1. **结构扫描**（快速）：目录结构、入口文件、路由、配置
2. **模块扫描**（按模块）：关键文件（入口、类型定义、主要逻辑）
3. **细节补充**（按需）：只在验证阶段发现遗漏时针对性读取

不要试图读完所有代码。20% 的文件包含 80% 的设计信息。

## 参数支持

从 `$ARGUMENTS` 解析范围。无参数时生成全部适用文档。
- `/autodev-reverse` — 全量逆向
- `/autodev-reverse ideation` — 只生成功能清单
- `/autodev-reverse api` — 只生成 API 设计
- `/autodev-reverse ui` — 只生成 UI 设计
- `/autodev-reverse design` — 只生成产品规格

## 反模式

| 禁止 | 应该 |
|------|------|
| 逐行翻译代码成文档 | 提炼设计意图和架构决策 |
| 试图读完每一个文件 | 读关键文件，从结构推断全貌 |
| 猜测不确定的内容 | 标注 `[推断]` 或 `[待确认]` |
| 生成的文档脱离 AutoDev 格式 | 严格遵循各阶段文档格式 |
| 覆盖用户已有的文档 | 检查 docs/plans/ 是否已有文档，提示选择 |
| 删除非文档文件 | 只建议删除 .md 文档，详见 `checklists/file-safety.md` |
