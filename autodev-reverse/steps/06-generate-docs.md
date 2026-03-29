# 步骤 6：生成文档

根据分析结果，生成 AutoDev 流水线格式的文档。

**生成哪些文档取决于项目类型：**

| 项目类型 | 生成的文档 |
|---------|-----------|
| 全栈项目 | ideation + design + ui + api |
| 纯前端 | ideation + design + ui |
| 纯后端/API | ideation + design + api |
| CLI 工具 | ideation + design |
| 库/SDK | ideation + design + api（接口文档） |

**文档格式严格遵循 AutoDev 流水线规范。** 各文档模板见 `templates/` 目录：
- `templates/ideation.md` — ideation 文档模板
- `templates/design.md` — design 文档模板

ui 文档遵循 `autodev-ui` 的输出格式：信息架构、页面清单、页面详设计、状态处理、数据需求汇总。

api 文档遵循 `autodev-api` 的输出格式：数据模型、端点清单、端点详设计、认证与授权、API 约定。

**文档标注规则：**
- 所有逆向生成的文档在标题下方标注 `> 来源：从现有代码库逆向生成`
- 推断出来的内容标注 `[推断]`，确认的内容不标注
- 不确定的地方标注 `[待确认]`，供用户审阅
