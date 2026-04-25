---
name: autodev-compress
description: 从设计文档生成地图式 index.md + 编码规则 rules.md，作为开发阶段的高效上下文载体。可独立使用，也被 autodev/autodev-add/autodev-reverse 自动调用。
metadata:
  promptSignals:
    phrases:
      - "autodev-compress"
      - "生成index"
      - "生成rules"
      - "压缩文档"
      - "生成地图"
---

# AutoDev Compress — 文档压缩为地图 + 规则

从设计文档中提取精炼的索引和规则文档，作为开发阶段的高效上下文载体。

## 设计理念

开发 subagent 不应加载全部设计文档（容易在长上下文中丢失关键约束）。
index.md 是**地图**（告诉你信息在哪），rules.md 是**规则**（始终加载的约束）。

## 使用场景

- **被编排器调用**：autodev Phase 5.5 / autodev-add Step 7 / autodev-reverse Step 6 自动调用
- **独立使用**：`/autodev-compress` 随时从现有设计文档重新生成 index + rules

## 输入

自动检测 `docs/plans/` 下的设计文档：
- `*-ideation.md`、`*-design.md`、`*-ui.md`、`*-api.md`、`*-plan.md`

## 产出 1：`*-index.md`（< 100 行）— 项目开发者地图

设计理念：这是一张**地图**，不是**摘要**。摘要浓缩信息（会丢失细节），地图告诉你信息在哪里（按需获取，不丢失）。

内容结构：
1. **项目概述**（1-2 句）：做什么、为谁做
2. **知识地图表**：每行一个知识领域，指向具体文件和章节

| 需要了解 | 去哪看 |
|---------|-------|
| 架构决策和技术栈 | → design.md §Architecture |
| 数据模型和关系 | → api.md §Data Models |
| 每个端点的详细规格 | → api.md §Endpoints / §{端点名} |
| 页面布局和组件清单 | → ui.md §{具体页面名} |
| 用户流程和状态转移 | → ui.md §User Flows |
| MVP 功能和优先级 | → ideation.md §MVP Summary |
| 编码约束和红线 | → rules.md（全文，< 80 行） |

（根据项目实际文档结构调整表格行，确保每个设计文档至少有一个指针入口）

3. **技术栈**（一行）：语言 | 框架 | 测试 | 包管理
4. **核心约束**（3-5 条，完整定义指向 rules.md）

**关键原则**：子 agent 拿到 index.md 后，应该**知道去哪找任何信息**，而不是**已经知道所有信息**。

## 产出 2：`*-rules.md`（< 80 行）— 编码规则手册

内容结构：
1. **技术栈约束**：框架版本、核心依赖版本号（从 design.md 技术选型表提取）
2. **编码约定**：命名规范、文件组织、组件拆分规则（从 design.md 提取或根据技术栈生成）
3. **API 约定**：认证方式、错误响应格式、分页规范（从 api.md 提取）
4. **质量红线**（精炼版）：
   - 禁止占位：无 TODO/pass/空函数体/硬编码空值
   - 禁止 Mock：无 mock/dummy/fake 数据替代真实调用
   - 禁止降阶：按设计文档方案实现，不替换为更简单的方案
   - 版本正确：依赖版本与技术选型表一致
5. **禁止模式**：从各设计文档中提取的具体 don'ts

## 生成规则
- 从已有设计文档中**提取**信息，不要凭空编造
- 严格控制行数：index < 100 行，rules < 80 行。超出时优先删除低优先级信息
- 如果设计文档中某个维度没有明确规定（如没有编码约定），该节留空并标注"待补充"

## 更新模式（被 autodev-add 调用时）

不重写整个 index/rules，而是：
- index.md：追加新功能对应的地图条目
- rules.md：追加新的约束（如果有）
- 验证行数仍在限制内
