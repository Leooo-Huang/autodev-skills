# 桩代码检测清单

对每个追踪项的关键代码文件做桩代码扫描：

| 桩代码信号 | 搜索模式 | 示例 |
|-----------|---------|------|
| 硬编码空值 | `= []`, `= {}`, `= null`, `= ''` 作为核心数据源 | `const holds: Hold[] = []` |
| TODO/FIXME 注释 | `// TODO`, `// FIXME`, `// HACK`, `// stub` | `// TODO: Load from route API` |
| Mock 数据 | 变量名含 `mock`, `dummy`, `fake`, `placeholder` | `const mockData = [...]` |
| 未调用的 API | 导入了 API 函数但从未在组件中调用 | `import { extractRoute }` 但搜不到 `extractRoute(` |
| 条件短路 | 核心逻辑被空数据跳过 | `if (holds.length === 0) return <Empty/>` 且 holds 永远为空 |

**规则**：
- 存在桩代码信号的功能，必须在备注中注明具体的桩代码位置
- 如果核心数据管道完全断裂（如数据源硬编码为空），应标为 `[ ]` 而非 `[~]`
- 核心数据管道断裂仍标 `[~]` 是**禁止**的
