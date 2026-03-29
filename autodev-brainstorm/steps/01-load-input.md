# 步骤 1：加载输入

**如果从流水线调用：**
- 读取 `docs/pipeline/state.yaml`，获取 `phases.1.output` 路径
- 读取该路径下的调研文档
- 读取 `docs/pipeline/config.yaml` 获取配置（`phases.brainstorm.*`）

**如果独立调用：**
- 查找 `docs/pipeline/research/` 下最新的调研文档
- 如果没有调研文档：警告用户，但仍可继续（降级为无调研的头脑风暴）

**同时读取项目上下文：**
- `CLAUDE.md`（如果存在）
- 包管理文件（`package.json` / `pyproject.toml` / `Cargo.toml` / `go.mod`）
- 最近 10 条 git log（`git log --oneline -10`）
- 顶层目录结构（`ls`）
