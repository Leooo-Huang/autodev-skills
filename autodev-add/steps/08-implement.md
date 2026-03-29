# 步骤 8：实现新功能

**目标**：按增量计划实现新功能代码。

## 执行

1. **加载上下文**：
   - 读取 `*-index.md` + `*-rules.md`（全局约束）
   - 读取步骤 6 的增量计划（当前任务）

2. **按计划逐项实现**：
   - 数据层：migration、model 定义
   - API 层：端点实现
   - UI 层：页面和组件
   - 调用 `frontend-design` skill 生成前端代码（如涉及 UI）

3. **质量红线**：
   - 参考 `autodev-shared/checklists/quality-redlines.md`
   - 禁止占位、禁止 Mock、禁止降阶、版本正确

4. **集成点特别注意**：
   - 修改现有文件时，只改必要的部分
   - 新增的 import/路由注册不要打乱现有代码结构

--- 步骤 8 完成 ---
