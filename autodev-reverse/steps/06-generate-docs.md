# 步骤 6：生成文档

根据分析结果，生成 AutoDev 流水线格式的文档。

**生成哪些文档取决于项目类型：**

| 项目类型 | 生成的文档 |
|---------|-----------|
| 全栈项目 | ideation + design + ui + api + index + rules |
| 纯前端 | ideation + design + ui + index + rules |
| 纯后端/API | ideation + design + api + index + rules |
| CLI 工具 | ideation + design + index + rules |
| 库/SDK | ideation + design + api（接口文档）+ index + rules |

**文档格式严格遵循 AutoDev 流水线规范。** 各文档模板见 `templates/` 目录：
- `templates/ideation.md` — ideation 文档模板
- `templates/design.md` — design 文档模板

ui 文档遵循 `autodev-ui` 的输出格式：信息架构、页面清单、页面详设计、状态处理、数据需求汇总。

api 文档遵循 `autodev-api` 的输出格式：数据模型、端点清单、端点详设计、认证与授权、API 约定。

## P/W 编号（与 autodev-ideation 对齐）

在 ideation 文档中，为每个功能分配 P/W 编号：

- **[P] 平台层功能**：底层能力、基础设施、核心模块。编号 P1, P2, P3...
- **[W] 场景层功能**：面向用户的业务场景、用例。编号 W1, W2, W3...
- 标注依赖关系：W 功能依赖哪些 P 功能

示例：
```
### MVP 功能清单
- [P1] 用户认证系统 ✅
- [P2] 数据存储层 ✅
- [W1] 搜索功能 (依赖 P2) ✅
- [W2] 结果导出 (依赖 P2, W1) ✅
```

## 生成 index.md（地图式，与 autodev Phase 5.5 对齐）

**必须生成** `*-index.md`（< 100 行），采用地图式格式：

```markdown
# {项目名} — 开发者地图

## 一句话
{做什么、为谁做}

## 知识地图
| 需要了解 | 去哪看 |
|---------|-------|
| 架构决策和技术栈 | → design.md §Architecture |
| 数据模型和关系 | → api.md §Data Models |
| 每个端点的详细规格 | → api.md §Endpoints |
| 页面布局和组件清单 | → ui.md §{页面名} |
| MVP 功能和优先级 | → ideation.md §MVP Summary |
| 编码约束和红线 | → rules.md（全文） |

## 技术栈
{语言} | {框架} | {测试} | {包管理}

## 核心约束（完整定义见 rules.md）
1. ...
2. ...
```

**关键原则**：index.md 是地图（告诉你信息在哪），不是摘要（浓缩信息）。

## 生成 rules.md（与 autodev Phase 5.5 对齐）

**必须生成** `*-rules.md`（< 80 行），从代码中提取实际遵循的规范：

1. **技术栈约束**：从 package.json/pyproject.toml 提取框架版本
2. **编码约定**：从代码中观察到的命名规范、文件组织、组件拆分模式
3. **API 约定**：从代码中观察到的认证方式、错误响应格式
4. **质量红线**（标准 4 条）：
   - 禁止占位：无 TODO/pass/空函数体
   - 禁止 Mock：无 mock/dummy/fake 数据
   - 禁止降阶：按设计文档方案实现
   - 版本正确：依赖版本与实际一致
5. **禁止模式**：从代码中观察到的具体 don'ts

## 更新 CLAUDE.md（必须）

生成文档后，**必须**更新项目根目录的 CLAUDE.md，在 `Design Docs`（或等价的文档索引段落）中加入指向新生成文档的指针。

CLAUDE.md 是"地图"——如果地图不指向新文档，未来会话中 Claude 无法发现它们，文档等于不存在。

**规则：**
1. 找到 CLAUDE.md 中列出设计文档/参考文档的段落
2. 追加新生成的 autodev 文档条目（ideation, design, index, rules，按实际生成的来）
3. 保持已有条目不动，只追加
4. 如果 CLAUDE.md 不存在该段落，新建一个 `## Design Docs` 段落

**示例追加内容：**
```markdown
- `docs/autodev-ideation.md` — 功能清单（AutoDev 格式）
- `docs/autodev-design.md` — 产品规格（AutoDev 格式）
- `docs/autodev-index.md` — 开发者地图
- `docs/autodev-rules.md` — 编码规则
```

## 文档标注规则
- 所有逆向生成的文档在标题下方标注 `> 来源：从现有代码库逆向生成`
- 推断出来的内容标注 `[推断]`，确认的内容不标注
- 不确定的地方标注 `[待确认]`，供用户审阅
