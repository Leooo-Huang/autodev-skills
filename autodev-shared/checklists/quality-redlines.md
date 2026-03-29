# 质量红线检查清单

## 红线 1：禁止占位实现
- [ ] 搜索代码无 `TODO`、`FIXME`、`HACK`、`XXX`、`stub` 注释
- [ ] 无 `pass`（Python）/空函数体（JS/TS）作为唯一语句
- [ ] 无 `NotImplementedError`
- [ ] 无硬编码空集合（`= []`/`= {}`/`= null`）作为核心数据源

## 红线 2：禁止 Mock 替代真实实现
- [ ] 搜索无 `mock`/`dummy`/`fake`/`placeholder`/`sample` 变量
- [ ] 设计文档中每个 API 在代码中有真实 HTTP 请求
- [ ] 无 `// 模拟数据`、`# fake data` 注释

## 红线 3：禁止降阶方案
- [ ] 代码使用的库/方案与 design.md 技术选型一致
- [ ] 计划中无 `for now`/`暂时`/`先用`/`简单起见`/`workaround`

## 红线 4：禁止使用过时技术
- [ ] `package.json`/`pyproject.toml` 版本与 design.md 标注一致
- [ ] 无已 deprecated 的 API 调用
