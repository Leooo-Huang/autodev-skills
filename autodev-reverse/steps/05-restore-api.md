# 步骤 5：还原 API 设计

从后端代码中还原 API 设计。

**分析内容：**

1. **数据模型**
   - 从 Schema/Model/Type 定义提取实体
   - 字段名、类型、约束
   - 实体间关系（外键、关联表）

2. **端点清单**
   - 从路由定义提取所有 API 端点
   - 方法、路径、中间件

3. **端点详情**
   - 从 handler/controller 代码提取：
     - 请求参数（path params、query params、body）
     - 响应格式（从 return/response 语句推断）
     - 业务规则（从 validation/if 逻辑推断）
     - 错误处理（从 catch/error 处理推断）

4. **认证与授权**
   - auth 中间件的应用范围
   - 权限检查逻辑

**跳过条件：** 如果项目没有 API（纯前端/静态站点），跳过此步骤。
