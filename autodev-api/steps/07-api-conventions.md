# 步骤 7：API 约定

定义统一的 API 风格约定，减少端点设计中的重复说明。

```
URL 风格：RESTful，小写，复数名词（/api/users, /api/items）
请求格式：JSON（Content-Type: application/json）
响应包装：{ "data": ..., "error": ..., "meta": ... }
分页：?page=1&limit=20，响应含 meta.total/meta.page/meta.limit
排序：?sort=created_at&order=desc
过滤：?status=active&type=premium
错误格式：{ "error": { "code": "ERROR_CODE", "message": "人类可读描述" } }
时间格式：ISO 8601（2024-01-15T08:30:00Z）
ID 格式：UUID v4
```
