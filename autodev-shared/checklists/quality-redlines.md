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

## 红线 5：优先复用开源方案
- [ ] `docs/plans/*-oss-scan.md` 存在（Phase 2 产出）
- [ ] `*-oss-scan.md` 中每个必需能力（R 编号）至少扫描了 3 个候选
- [ ] `*-design.md` 含"复用决策表"，每个 R 能力标注了"复用 Lib-X / 自研"
- [ ] 所有自研的 R 能力在 design.md 中有显式"自研理由"章节，对比候选方案列出差距
- [ ] 不得跳过 `/last30days` 扫描（除非运行在离线环境且已在 state.yaml 标注 fallback）

## 红线 6：前端禁用 emoji 作为 UI 图标
- [ ] `*-ui.md` 视觉规范章节第 6 项"图标系统"已填写（图标库名、风格、尺寸、描边粗细）
- [ ] `package.json` / `requirements` 中已引入 ui.md 指定的 icon 库（如 `lucide-react`、`@heroicons/react`、`@phosphor-icons/react` 等）
- [ ] 前端代码（`*.tsx/*.jsx/*.vue/*.svelte`）的 JSX/template text 节点中无 emoji unicode 字符
- [ ] 用于功能图标的元素使用 icon 库组件（如 `<Trash2 />`、`<XMarkIcon />`），不用 `<span>🗑️</span>`
- [ ] Emoji 仅出现在以下白名单场景：i18n 文案 JSON 的值、代码注释、UGC 内容、README 等文档
- [ ] 状态指示（成功/失败/警告/loading/empty）使用 icon 库 + 颜色 token，不用 ✅❌⚠️⏳📭 等 emoji 替代

## 红线 7：禁止未验证即断言环境不可用（反自导自演降阶）

**触发判断**：当文档/代码/对话中出现以下短语时必须同时附带对应的验证命令实际输出：

- "环境没有 Node / pnpm / npm / git / psql / docker / vercel / gh"
- "跑不了 build / install / typecheck / test / dev / migrate"
- "无法执行 / 运行 pnpm build / typecheck"
- "this environment has no X" / "cannot run X" / "no X available"
- "由于没有 X，所以跳过 Y"

**必须先跑的验证命令（对应关系）**：

| 断言 | 允许使用的证据 |
|------|--------------|
| 没有 Node | `node --version` 失败或 exit ≠ 0 |
| 没有 pnpm | `pnpm --version` 失败 |
| 没有 npm | `npm --version` 失败 |
| 没有 Postgres | `which psql` 无输出 **且** `pg_isready` 失败 |
| 没有 git repo | `git rev-parse --is-inside-work-tree` 失败 |
| 跑不了 build | 实际执行过 `pnpm build`（或等价命令）并记录真实错误 |
| 没有 Vercel CLI | `vercel --version` 失败 |
| 没有 GitHub CLI | `gh --version` 失败 |
| 没有 Docker | `docker --version` 失败 |

- [ ] 所有"环境不可用"断言都附带了验证命令的实际输出/exit code
- [ ] 自己写的 plan 中的 "MVP 范围 / 延期到 V0.X" **未被当作外部约束来合理化降阶**
- [ ] 进入 sleep / ralph / autonomous 模式前，`docs/pipeline/env-capabilities.yaml` 已产出，包含所有声明依赖的工具版本和状态
- [ ] 任何"跳过"决定都能在 env-capabilities.yaml 中找到对应的真实探测结果

**特别提醒**：
- 如果 plan 是你自己写的，它里面的"延期"不是外部约束，是你给自己找的借口。看真实代码阻塞，不看自己写的范围。
- "token 不够" / "context 不够" 不是环境不可用的代名词——那是 context 预算问题，需要另外声明，不得混淆。
