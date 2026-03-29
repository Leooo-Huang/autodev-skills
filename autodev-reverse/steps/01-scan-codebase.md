# 步骤 1：扫描代码库

快速建立对项目的整体认知。

**扫描内容：**

1. **项目元数据**
   - `package.json` / `Cargo.toml` / `pyproject.toml` / `go.mod` 等 → 技术栈、依赖
   - `.env.example` / 配置文件 → 外部服务依赖
   - `README.md` / 已有文档 → 项目描述（可能过时，仅作参考）

2. **目录结构**
   - `ls` 顶层目录，理解项目组织方式
   - 识别关键目录：`src/`、`pages/`、`routes/`、`api/`、`components/`、`models/`、`tests/`

3. **Git 历史**（如有）
   - `git log --oneline -30` → 最近的开发活动
   - `git log --oneline --all | wc -l` → 项目规模感知
   - `git log --format="%s" | head -50` → 从提交信息理解功能演进

4. **代码统计**
   - 文件数量和类型分布
   - 主要语言/框架

**产出：项目概况**
- 技术栈（语言、框架、数据库、外部服务）
- 项目规模（文件数、代码量级）
- 目录结构概览
