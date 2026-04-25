# 步骤 1：加载设计系统 & 全站审计

**目标**：确保 DESIGN.md 存在，提取结构化 token，截图全站，建立 baseline 评分。

## 1.1 确认 DESIGN.md

检查项目根目录是否存在 `DESIGN.md`。

**如果存在** → 读取它，进入 1.2。

**如果不存在** → 引导用户获取：

```
项目根目录没有 DESIGN.md。你需要一个设计系统作为视觉真值标准。

获取方式：
1. 从 awesome-design-md 安装品牌设计系统：
   npx getdesign@latest add <brand>
   可选品牌：stripe / linear / vercel / notion / figma / claude 等 60+ 个

2. 自己编写（参考 https://github.com/VoltAgent/awesome-design-md 的 9 章结构）

请选择一个品牌，或提供你自己的 DESIGN.md。
```

等待用户操作后，再次检查 `DESIGN.md` 是否存在。**没有 DESIGN.md 不能继续。**

## 1.2 提取结构化 Token

读取 DESIGN.md，提取以下信息写入 `polish/design-tokens.md`：

```markdown
# Design Tokens（从 DESIGN.md 提取）

## 颜色
| 角色 | 色值 | 用途 |
|------|------|------|
| primary | #xxx | 按钮、链接、焦点 |
| background | #xxx | 页面背景 |
| surface | #xxx | 卡片、面板背景 |
| foreground | #xxx | 主要文字 |
| muted | #xxx | 次要文字 |
| border | #xxx | 分割线、边框 |
| accent | #xxx | 强调元素 |
| destructive | #xxx | 错误、删除 |

## 字体
| 层级 | 字体族 | 字号 | 字重 |
|------|--------|------|------|
| Display | xxx | xx | xx |
| H1 | xxx | xx | xx |
| Body | xxx | xx | xx |
...

## 间距
基础单位：{N}px
刻度：{...}

## 圆角
| 元素 | 圆角值 |
|------|--------|
| 按钮 | xx |
| 卡片 | xx |
...

## 阴影
| 层级 | 阴影值 |
|------|--------|
...

## Do's and Don'ts（摘要）
- Do: ...
- Don't: ...
```

如果 DESIGN.md 中某些章节缺失，记录缺失项，但不阻塞流程。

## 1.3 确认 Dev Server 运行

检查 dev server 是否在运行（检测常见端口：3000, 5173, 8080, 4200）。

**如果未运行** → 提示用户启动：
```
请先启动 dev server。常见命令：
- npm run dev
- pnpm dev
- yarn dev
```

## 1.4 检测截图工具

按优先级检测：
1. chrome-devtools MCP（`take_screenshot`）
2. playwright MCP（`browser_take_screenshot`）

记录可用的工具。**两个都不可用 → 警告用户，但不阻塞**（降级为纯代码审查模式）。

## 1.5 发现所有页面

通过以下方式发现项目的所有页面路由：
1. 读取路由配置文件（`app/` 目录结构 / `routes.ts` / `router.tsx` 等）
2. 如果无法自动发现 → 请用户列出所有页面 URL

产出页面清单：
```markdown
# 页面清单
| # | 路由 | slug | 说明 |
|---|------|------|------|
| 1 | / | home | 首页 |
| 2 | /dashboard | dashboard | 仪表盘 |
...
```

## 1.6 截图全站 & Baseline 评分

对每个页面：
1. 导航到页面 URL
2. 截图保存（记录路径）
3. **Spawn 独立 Evaluator agent** 做 baseline 评分

Evaluator agent 的 prompt 模板：

```
你是一个严格的视觉设计评审。

你的任务：评估这个页面截图与给定设计系统的匹配程度。

设计系统 token：
{design-tokens.md 内容}

评分标准：
{evaluator-rubric.md 内容}

请查看截图并评分。输出格式严格遵循 scorecard 模板。
注意：你的角色是找出问题，不是表扬。"还行"不是一个可接受的评价。
```

将所有页面的 baseline 评分汇总写入 `polish/baseline-report.md`：

```markdown
# Baseline 评估报告

| 页面 | Compliance | Craft | Coherence | Functionality | 总分 | 状态 |
|------|-----------|-------|-----------|--------------|------|------|
| home | 2 | 3 | 2 | 4 | 11/20 | NEEDS_WORK |
| dashboard | 1 | 2 | 2 | 4 | 9/20 | NEEDS_WORK |
...

## 优先级排序（从低分到高分）
1. dashboard (9/20) — 最需要改进
2. home (11/20)
...
```

## 1.7 用户确认

展示 baseline 报告给用户，确认：
- 页面清单是否完整
- 优先级排序是否合理
- 是否有页面要跳过

--- 步骤 1 完成 ---
