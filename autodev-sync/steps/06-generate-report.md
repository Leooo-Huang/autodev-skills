# 步骤 5：生成同步报告

在终端展示简洁的进度摘要，格式参见 `templates/sync-report.md`。

也写入 `docs/pipeline/sync-report.md` 留档。

## 批次追踪

支持分批交付的场景：

```
/autodev-sync              — 同步全部文档
/autodev-sync mvp          — 只同步 MVP 范围的功能
/autodev-sync v2           — 只同步 V2 范围的功能
```

从 `$ARGUMENTS` 解析批次范围。无参数时同步全部。
