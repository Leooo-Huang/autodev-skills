---
name: autodev
description: "全自动开发流水线：想法 → 产品创意 → 产品规格 → UI/UX → API → 规划 → 编码 → 交付。当用户说 autodev、auto-dev、自动开发 或需要端到端自动化开发时使用。"
user-invocable: true
allowed-tools: [Agent, Bash, Read, Write, Skill, TodoWrite, AskUserQuestion]
---

# AutoDev — 全自动开发流水线

从一个想法或文档出发，自动走完 8 个阶段的全流程。通过配置文件控制门控，默认全自动。

| 阶段 | 名称 | 执行方式 | 产出 |
|------|------|---------|------|
| 1 | 产品创意 | `autodev-ideation` 技能 | `*-ideation.md` |
| 2 | 产品规格 | MECE 分解 + `superpowers:brainstorming` | `*-design.md` |
| 3 | UI/UX 设计 | `autodev-ui` 技能 | `*-ui.md` |
| 4 | API 设计 | `autodev-api` 技能 | `*-api.md` |
| 5 | 规划 | `superpowers:writing-plans` | `*-plan.md` |
| 5.5 | 索引与规则 | 内置提取 | `*-index.md` + `*-rules.md` |
| 6 | 开发 | `superpowers:subagent-driven-development` | 代码提交 |
| 7 | 验证 | `acceptance-testing` 技能 | 测试通过 |
| 8 | 交付 | `superpowers:finishing-a-development-branch` + 部署 | 合并/PR |

**文档链：** `ideation.md → design.md → ui.md → api.md → plan.md → index.md + rules.md → 代码 → 验证 → 交付`

## 质量红线

4 条红线贯穿全流水线（详见 `autodev-shared/checklists/quality-redlines.md`）：
1. **禁止占位** — 无 TODO/pass/空函数体
2. **禁止 Mock** — 无 mock/dummy/fake 数据替代真实调用
3. **禁止降阶** — 按设计文档方案实现
4. **禁止过时** — WebSearch 验证版本号

## 何时使用

使用：用户说 "autodev"、"自动开发"、"从头开始做一个 X"
不使用：只需单阶段时用对应子技能（`/autodev-ideation`、`/autodev-ui`、`/autodev-api`）

## 步骤索引

| 步骤 | 文件 | 内容 |
|------|------|------|
| 1 | `steps/01-initialize.md` | 解析输入、检查状态、创建配置和 state.yaml |
| 2 | `steps/02-phase-dispatch.md` | PRE/RUN/POST/GATE 通用调度协议 |
| 3 | `steps/03-phase-1-ideation.md` | 阶段 1：产品创意 |
| 4 | `steps/04-phase-2-spec.md` | 阶段 2：MECE 分解 + 产品规格 |
| 5 | `steps/05-phase-3-ui.md` | 阶段 3：UI/UX 设计 |
| 6 | `steps/06-phase-4-api.md` | 阶段 4：API 设计 |
| 7 | `steps/07-phase-5-planning.md` | 阶段 5：规划（含降阶扫描） |
| 8 | `steps/08-phase-5.5-index-rules.md` | 阶段 5.5：生成 index.md + rules.md |
| 9 | `steps/09-phase-6-development.md` | 阶段 6：开发（subagent 上下文 + 红线传递） |
| 10 | `steps/10-phase-7-verification.md` | 阶段 7：红线扫描 + acceptance-testing |
| 11 | `steps/11-phase-8-delivery.md` | 阶段 8：代码集成 + hooks 推荐 + 部署 |
| 12 | `steps/12-resume-protocol.md` | 恢复协议 |
| 13 | `steps/13-sleep-mode.md` | 睡眠模式（Ralph Loop） |
| 14 | `steps/14-error-handling.md` | 错误处理（失败/跳过/中止） |

## 执行规则

<IMPORTANT>
按步骤顺序执行。执行步骤 N 时，用 Read 工具读取对应的 steps/ 文件获取详细指令。
只加载当前步骤的文件，不要一次性读取所有步骤——这是为了节省 token 并保持专注。
每步完成后输出 `--- 步骤 N 完成 ---`。
共享资源在 `autodev-shared/` 目录下，按需读取。
</IMPORTANT>

## 反模式

| 禁止 | 应该 |
|------|------|
| 跳过门控检查逻辑 | 始终执行（自动模式下自动通过） |
| 协议外修改 state.yaml | 只在 PRE/POST/GATE/错误处理时更新 |
| 不验证输出就启动下一阶段 | POST 中验证输出文件存在且非空 |
| 把全部设计文档塞进 subagent | 只传 index.md + rules.md + 当前任务章节 |
| 验证未通过就进入交付 | 失败时回到阶段 6 修复 |
| 睡眠模式不设 max_iterations | 始终有安全上限 |

## 快速参考

```
/autodev "做一个攀岩 app"           # 全新流水线
/autodev "做一个攀岩 app" --sleep   # 睡眠模式
/autodev --resume                   # 从断点恢复
/autodev --skip-phase 1             # 跳过阶段
/autodev --abort                    # 中止流水线
```
