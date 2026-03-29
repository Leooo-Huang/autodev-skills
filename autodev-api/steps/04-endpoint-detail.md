# 步骤 4：端点详细设计

对每个端点定义完整的请求/响应规格。

**格式：**

```
### {METHOD} {path}

**用途：** {做什么}
**对应 UI：** {哪个页面的什么操作}

**请求参数：**
- Path: `id` (string, required) — 资源 ID
- Query: `page` (int, optional, default=1) — 分页
- Query: `limit` (int, optional, default=20, max=100) — 每页数量
- Body:
  ```json
  {
    "name": "string (required, 1-100 chars)",
    "description": "string (optional, max 500 chars)"
  }
  ```

**成功响应：** (200/201)
```json
{
  "data": { ... },
  "meta": { "total": 42, "page": 1, "limit": 20 }
}
```

**错误响应：**
| 状态码 | 错误码 | 场景 |
|--------|--------|------|
| 400 | VALIDATION_ERROR | 请求参数不合法 |
| 404 | NOT_FOUND | 资源不存在 |
| 403 | FORBIDDEN | 无权限 |

**业务规则：**
- {规则 1：如"只能删除自己创建的"}
- {规则 2：如"名称不能重复"}
```

**关键要求：**
- 每个端点都要有错误响应定义——不只是"成功时返回什么"
- 业务规则要具体：不写"需要验证"，写"名称长度 1-100 字符，不能包含特殊字符"
- 列表端点统一分页格式
