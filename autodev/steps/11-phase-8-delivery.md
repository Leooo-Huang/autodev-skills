# 步骤 11：阶段 8 — 交付与部署

**目标**：代码集成、生成 hooks 推荐、可选部署。

## 第一步：代码集成

调用 `superpowers:finishing-a-development-branch` 技能。
自动模式下：使用 `delivery.default_action` 配置项决定动作（ask/merge/pr/keep）。

## 第 1.5 步：生成推荐 Hooks 配置

读取 `*-rules.md`，为关键约束生成 Claude Code hooks 配置建议。

| rules.md 中的规则类型 | 推荐 Hook 类型 | 示例 |
|---|---|---|
| 提交前检查 | PreCommit | `npm test`、`npm run lint` |
| 文件保护 | PreToolCall(Edit) | 检查文件路径不含 `migrations/`、`*.lock` |
| 代码格式化 | PostToolCall(Edit) | `prettier --write {file}` |
| 构建验证 | PostToolCall(Write) | `npx tsc --noEmit` |

**产出**：展示可粘贴的 hooks 配置片段 + 写入 `docs/pipeline/recommended-hooks.json`。
**注意**：这只是推荐，**不自动修改 settings.json**。

## 第二步：部署（可选）

读取 `*-design.md` 中的「部署策略」章节。

| 平台 | 调用方式 |
|----|--------|
| Vercel | `vercel:deploy` 插件 |
| Railway | `railway` 技能 |
| Coolify | `coolify-manager` 技能 |
| Docker/VPS | Bash 执行部署脚本 |
| 未指定 | 跳过部署 |

<IMPORTANT>
部署是可选步骤。design.md 没有部署策略、技能未安装、或配置关闭了自动部署，都优雅跳过。部署失败不影响交付完成状态——代码集成成功即算交付完成。
</IMPORTANT>

--- 步骤 11 完成 ---
