# 同步报告模板

```
同步完成 — {项目名} ({日期})

功能进度：
  MVP:    3/5 完成 (60%)  [██████░░░░]
  V2:     0/3 完成 (0%)   [░░░░░░░░░░]

页面进度：  4/7 已实现
API 进度：  8/12 已实现
任务进度：  6/15 已完成

未记录功能（代码有，文档没有）：
  - src/utils/analytics.ts（分析工具，建议补充到 design 文档）

状态过时（文档和代码不一致）：
  - 无

设计漂移检测：
  ERROR (0):
    - (无)
  WARN (2):
    - design.md 选型 prisma@5.0 → 实际 prisma@6.1（package.json）
    - api.md 定义 GET /api/users → 代码中不存在该路由
  INFO (3):
    - 新依赖 zod 未记录在 design.md
    - 新端点 POST /api/webhooks 未记录在 api.md
    - 新页面 /settings/notifications 未记录在 ui.md

文档摘要一致性（index.md / rules.md）：
  - index.md 与设计文档一致性：✓
  - rules.md 与设计文档一致性：⚠️ 技术选型表版本号未同步（prisma@5.0 → 6.1）

已更新文档：
  - docs/plans/*-ideation.md（添加状态列）
  - docs/plans/*-api.md（更新 3 个端点状态）
```

也写入 `docs/pipeline/sync-report.md` 留档。
