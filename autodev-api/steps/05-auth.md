# 步骤 5：认证与授权

定义系统的认证和权限模型。

**需要回答的问题：**
- 认证方式？（JWT / Session / OAuth / API Key）
- 哪些端点需要认证？哪些是公开的？
- 有没有不同的用户角色？权限有什么差异？
- Token 的生命周期（有效期、刷新机制）

**格式：**
```
认证方式：{JWT / Session / ...}
Token 位置：{Authorization header / Cookie / ...}

权限矩阵：
| 端点 | 未登录 | 普通用户 | 管理员 |
|------|--------|---------|--------|
| GET /api/items | ✓ | ✓ | ✓ |
| POST /api/items | ✗ | ✓ | ✓ |
| DELETE /api/items/:id | ✗ | 仅自己的 | ✓ |
```
