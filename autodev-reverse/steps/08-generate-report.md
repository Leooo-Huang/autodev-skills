# 步骤 8：生成逆向报告

在终端展示摘要，格式参见 `templates/reverse-report.md`。

也写入 `docs/pipeline/reverse-report.md` 留档。

## 参数支持

```
/autodev-reverse              — 全量逆向，生成所有适用的文档
/autodev-reverse ideation     — 只生成功能清单文档
/autodev-reverse api          — 只生成 API 设计文档
/autodev-reverse ui           — 只生成 UI 设计文档
/autodev-reverse design       — 只生成产品规格文档
```

从 `$ARGUMENTS` 解析范围。无参数时生成全部适用文档。
