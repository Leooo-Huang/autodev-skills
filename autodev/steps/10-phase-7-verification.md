# 步骤 10：阶段 7 — 验证

**目标**：执行红线扫描 + acceptance-testing，确保代码质量和功能完整性。

## 红线扫描（在功能验证之前执行）

<IMPORTANT>
扫描不通过则直接回退到 Phase 6，不进入功能测试。详细扫描项参考 `autodev-shared/checklists/quality-redlines.md`。
</IMPORTANT>

**扫描 1：占位代码（红线 1）** — 搜索项目代码（排除 node_modules、.venv、dist 等）：`TODO`、`FIXME`、`HACK`、`XXX`、`stub`、`pass`（Python）、空函数体、`NotImplementedError`、硬编码空集合作为核心数据源

**扫描 2：Mock 数据（红线 2）** — 搜索含 `mock`/`dummy`/`fake`/`placeholder`/`sample` 的数据定义；对照 `*-api.md` 检查每个端点是否有真实 HTTP 请求

**扫描 3：降阶实现（红线 3）** — 对照 `*-design.md` 技术选型，检查代码中使用的库/方案是否一致

**扫描 4：版本一致性（红线 4）** — 对照设计文档技术选型表，检查 `package.json`/`pyproject.toml` 中的依赖版本

**扫描结果处理：**
- 发现问题 → 列出具体位置和违反的红线编号 → 回退到 Phase 6 修复
- 全部通过 → 继续 acceptance-testing

## 功能验证

调用 `acceptance-testing` 技能。

**衔接：** 读取 `*-ui.md`（页面清单、用户流程）和 `*-api.md`（端点清单）作为验证的功能目录输入。

- **输入**：已提交的代码 + `*-ui.md` + `*-api.md`
- **产出**：全部测试通过（或修复后通过）
- **阻塞规则**：如果验证未通过，不得进入阶段 8。失败时回到阶段 6 修复代码，再重新验证。

--- 步骤 10 完成 ---
