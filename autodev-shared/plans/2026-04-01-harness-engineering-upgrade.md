# Harness Engineering 全面升级实施计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 将 AutoDev skill 系统从纯 skill 驱动升级为 skill + hook 混合 harness，实现一句话触发全自动开发。

**Architecture:** 3 层改动：Layer 1 全局 hooks 基础设施（新增）→ Layer 2 autodev pipeline 核心升级（修改）→ Layer 3 跨 session 连续性（修改 sleep mode）。确定性约束（hooks）兜底，概率性方法论（skills）指导。

**Tech Stack:** Node.js hooks 脚本 | Markdown skill 文件 | YAML 状态管理

**改动范围：** 新增 7 个文件，修改 8 个文件，不动 113 个文件。

---

## Task 1: 创建全局 Hook 脚本 — session-start.js

**Files:**
- Create: `~/.claude/hooks/session-start.js`

- [ ] **Step 1: 创建 hooks 目录**

```bash
mkdir -p ~/.claude/hooks
```

- [ ] **Step 2: 编写 session-start.js**

```javascript
// ~/.claude/hooks/session-start.js
// SessionStart hook: 检测新项目/断点恢复 → 注入 context
const fs = require('fs');
const path = require('path');

const cwd = process.cwd();

// 1. 检测进行中的 pipeline（autodev 断点恢复）
const stateFile = path.resolve(cwd, 'docs/pipeline/state.yaml');
const handoffFile = path.resolve(cwd, '.claude/handoff.md');

if (fs.existsSync(handoffFile)) {
  const content = fs.readFileSync(handoffFile, 'utf8');
  const firstLines = content.split('\n').slice(0, 15).join('\n');
  process.stderr.write(
    `\n⚠️ 检测到 pipeline 交接文档。请读取 .claude/handoff.md 获取精确进度并继续执行。\n` +
    `交接摘要:\n${firstLines}\n`
  );
} else if (fs.existsSync(stateFile)) {
  const content = fs.readFileSync(stateFile, 'utf8');
  if (content.includes('in_progress')) {
    process.stderr.write(
      `\n⚠️ 检测到进行中的 autodev pipeline。请调用 /autodev --resume 从断点继续。\n`
    );
  }
}

// 2. 检测新项目（无 CLAUDE.md 但有项目文件）
const hasClaudeMd = fs.existsSync(path.resolve(cwd, 'CLAUDE.md'));
const hasPackageJson = fs.existsSync(path.resolve(cwd, 'package.json'));
const hasPyproject = fs.existsSync(path.resolve(cwd, 'pyproject.toml'));
const hasGoMod = fs.existsSync(path.resolve(cwd, 'go.mod'));
const hasCargo = fs.existsSync(path.resolve(cwd, 'Cargo.toml'));

if (!hasClaudeMd && (hasPackageJson || hasPyproject || hasGoMod || hasCargo)) {
  process.stderr.write(
    `\n⚠️ 检测到项目文件但无 CLAUDE.md。建议运行 /harness-init 为此项目搭建 harness 结构。\n`
  );
}
```

- [ ] **Step 3: 验证脚本语法**

```bash
node -c ~/.claude/hooks/session-start.js
```

Expected: 无输出（语法正确）

---

## Task 2: 创建全局 Hook 脚本 — post-edit-check.js

**Files:**
- Create: `~/.claude/hooks/post-edit-check.js`

- [ ] **Step 1: 编写 post-edit-check.js**

```javascript
// ~/.claude/hooks/post-edit-check.js
// PostToolUse hook: 每次 Edit/Write 后自动检查
const fs = require('fs');
const path = require('path');
const { execSync } = require('child_process');

let input = '';
process.stdin.setEncoding('utf8');
process.stdin.on('data', d => input += d);
process.stdin.on('end', () => {
  try {
    const ctx = JSON.parse(input);
    const file = ctx.tool_input?.file_path || ctx.tool_input?.path || '';
    if (!file) return;

    const cwd = process.cwd();
    const errors = [];

    // TypeScript: 编辑 .ts/.tsx 文件后跑 type check
    if (/\.tsx?$/.test(file)) {
      const tsconfigPaths = [
        path.resolve(cwd, 'tsconfig.json'),
        path.resolve(cwd, 'ts/tsconfig.json'),
      ];
      const tsconfig = tsconfigPaths.find(p => fs.existsSync(p));
      if (tsconfig) {
        const tsDir = path.dirname(tsconfig);
        try {
          execSync('npx tsc --noEmit 2>&1', { timeout: 30000, cwd: tsDir });
        } catch (e) {
          const output = (e.stdout || '').toString().slice(0, 800);
          if (output.trim()) {
            errors.push(`TypeScript 类型错误:\n${output}`);
          }
        }
      }
    }

    // Python: 编辑 .py 文件后跑 ruff
    if (file.endsWith('.py') && fs.existsSync(path.resolve(cwd, 'pyproject.toml'))) {
      try {
        execSync(`ruff check "${file}" 2>&1`, { timeout: 10000, cwd });
      } catch (e) {
        const output = (e.stdout || '').toString().slice(0, 500);
        if (output.trim()) {
          errors.push(`Python lint 错误:\n${output}`);
        }
      }
    }

    // 红线 1 实时化: 扫描刚写入的文件是否含占位符
    if (/\.(ts|tsx|js|jsx|py|go|rs)$/.test(file) && fs.existsSync(file)) {
      const content = fs.readFileSync(file, 'utf8');
      const placeholders = [];
      const lines = content.split('\n');
      lines.forEach((line, i) => {
        if (/\b(TODO|FIXME|HACK|XXX)\b/.test(line)) {
          placeholders.push(`  L${i + 1}: ${line.trim().slice(0, 80)}`);
        }
      });
      if (placeholders.length > 0) {
        errors.push(`占位符警告（红线 1）:\n${placeholders.slice(0, 5).join('\n')}`);
      }
    }

    if (errors.length > 0) {
      process.stderr.write(`\n⚠️ 编辑后自动检查发现问题:\n${errors.join('\n\n')}\n请修复后继续。\n`);
    }
  } catch (e) {
    // 静默失败，不阻塞 Claude 工作流
  }
});
```

- [ ] **Step 2: 验证语法**

```bash
node -c ~/.claude/hooks/post-edit-check.js
```

Expected: 无输出（语法正确）

---

## Task 3: 创建全局 Hook 脚本 — session-stop.js

**Files:**
- Create: `~/.claude/hooks/session-stop.js`

- [ ] **Step 1: 编写 session-stop.js**

```javascript
// ~/.claude/hooks/session-stop.js
// Stop hook: 提醒保存交接状态
const fs = require('fs');
const path = require('path');

const cwd = process.cwd();
const stateFile = path.resolve(cwd, 'docs/pipeline/state.yaml');

if (fs.existsSync(stateFile)) {
  const content = fs.readFileSync(stateFile, 'utf8');
  if (content.includes('in_progress')) {
    process.stderr.write(
      '\n💾 检测到 autodev pipeline 仍在进行中。' +
      '如果即将断开，请确保已生成 .claude/handoff.md 交接文档。\n'
    );
  }
}
```

- [ ] **Step 2: 验证语法**

```bash
node -c ~/.claude/hooks/session-stop.js
```

Expected: 无输出

---

## Task 4: 注册 Hooks 到全局 settings.json

**Files:**
- Modify: `~/.claude/settings.json`

- [ ] **Step 1: 在 settings.json 中添加 hooks 配置**

在现有 JSON 结构中添加 `"hooks"` 字段（与 `"env"`、`"permissions"` 同级）：

```json
"hooks": {
  "SessionStart": [{
    "matcher": "startup|resume",
    "hooks": [{
      "type": "command",
      "command": "node C:\\Users\\Leo23\\.claude\\hooks\\session-start.js"
    }]
  }],
  "PostToolUse": [{
    "matcher": "Edit|Write",
    "hooks": [{
      "type": "command",
      "command": "node C:\\Users\\Leo23\\.claude\\hooks\\post-edit-check.js"
    }]
  }],
  "Stop": [{
    "matcher": "",
    "hooks": [{
      "type": "command",
      "command": "node C:\\Users\\Leo23\\.claude\\hooks\\session-stop.js"
    }]
  }]
}
```

- [ ] **Step 2: 验证 settings.json 仍然是有效 JSON**

```bash
node -e "JSON.parse(require('fs').readFileSync(require('os').homedir() + '/.claude/settings.json', 'utf8')); console.log('Valid JSON')"
```

Expected: `Valid JSON`

---

## Task 5: 创建 harness-init Skill

**Files:**
- Create: `~/.claude/skills/harness-init/SKILL.md`
- Create: `~/.claude/skills/harness-init/templates/rules-security.md`
- Create: `~/.claude/skills/harness-init/templates/rules-no-placeholders.md`

- [ ] **Step 1: 创建目录结构**

```bash
mkdir -p ~/.claude/skills/harness-init/templates
```

- [ ] **Step 2: 编写 SKILL.md**

写入 `~/.claude/skills/harness-init/SKILL.md`，内容为完整的 harness-init skill 定义（见设计方案中的详细内容）。

核心执行流程：
1. 扫描项目（package.json/tsconfig/pyproject.toml/go.mod/Cargo.toml）
2. 让用户确认项目信息
3. 生成地图式 CLAUDE.md（≤60 行）
4. 生成 .claude/rules/（security + no-placeholders + testing + code-style）
5. 确认并 git add

- [ ] **Step 3: 创建 rules 模板 — security.md**

写入 `~/.claude/skills/harness-init/templates/rules-security.md`：
- 禁止硬编码敏感信息
- 环境变量获取
- .env 在 .gitignore
- 禁止日志输出敏感信息
- 禁止 force push main/master

- [ ] **Step 4: 创建 rules 模板 — no-placeholders.md**

写入 `~/.claude/skills/harness-init/templates/rules-no-placeholders.md`：
- 禁止 TODO/FIXME/HACK/XXX/stub
- 禁止 pass/空函数体
- 禁止 NotImplementedError
- 禁止硬编码空集合
- 禁止 mock/dummy/fake/placeholder/sample 变量

---

## Task 6: 修改 Phase 5.5 — index.md 改为地图式

**Files:**
- Modify: `~/.claude/skills/autodev/steps/08-phase-5.5-index-rules.md`

- [ ] **Step 1: 替换"产出 1"章节**

将原来的摘要式模板替换为地图式模板。关键变化：

原来：`文档清单`、`关键设计决策`、`能力-组件映射表`、`MVP 功能清单`
改为：`知识地图表`（每行 = 需要了解什么 → 去哪看 file §section）

保留"产出 2"（rules.md）和"生成规则"不变。

在"产出 1"章节开头加设计理念说明：
> 这是一张**地图**，不是**摘要**。摘要浓缩信息（会丢失细节），地图告诉你信息在哪里（按需获取，不丢失）。

---

## Task 7: 修改 Phase 5 — plan.md 加 acceptance criteria

**Files:**
- Modify: `~/.claude/skills/autodev/steps/07-phase-5-planning.md`

- [ ] **Step 1: 在"执行"和"计划质量扫描"之间插入新章节**

插入 `## 契约式验收标准（Harness Engineering）` 章节：
- 要求每个 task 包含 `acceptance_criteria` 字段（3-5 条可客观验证的标准）
- 要求包含 `status: pending` 字段
- 列出禁止的模糊标准示例
- 列出正确的具体标准示例

---

## Task 8: 修改 Phase 6 — 加独立 reviewer + 活文档

**Files:**
- Modify: `~/.claude/skills/autodev/steps/09-phase-6-development.md`

- [ ] **Step 1: 在"前端代码质量规则"之后添加"独立审查"章节**

插入 `## 独立审查（Harness Engineering — 生成与评估分离）` 章节：
- 每个 task 完成后 spawn 独立 reviewer agent
- reviewer prompt 包含 acceptance_criteria + 4 个检查维度
- 输出格式：VERDICT (PASS/FAIL) + DETAILS + SUGGESTIONS
- 最多 3 轮 review，FAIL 反馈给原 agent 修复
- 3 轮后仍 FAIL → 记录问题继续（不阻塞）

- [ ] **Step 2: 添加"活文档更新"章节**

插入 `## 活文档更新（Plan as Living Document）` 章节：
- reviewer PASS 后更新 plan.md：status → completed
- 偏离决策记录在 `decision:` 字段
- 示例展示完整的更新后 task 格式

---

## Task 9: 修改 Phase 7 — 从大爆炸扫描改为契约确认

**Files:**
- Modify: `~/.claude/skills/autodev/steps/10-phase-7-verification.md`

- [ ] **Step 1: 重写文件**

新的 Phase 7 三层验证：
1. **契约验收**（新增）：对照 plan.md 的 acceptance_criteria + reviewer 记录
2. **红线最终确认**（简化）：兜底扫描，因为 hook 已实时拦截
3. **功能验证**（不变）：acceptance-testing skill

---

## Task 10: 修改 Sleep Mode — 加 handoff.md 生成

**Files:**
- Modify: `~/.claude/skills/autodev/steps/13-sleep-mode.md`
- Create: `~/.claude/skills/autodev-shared/templates/handoff.md`

- [ ] **Step 1: 创建 handoff 模板文件**

写入 `~/.claude/skills/autodev-shared/templates/handoff.md`，包含：
- 当前位置（Phase + Task 编号）
- 已完成列表
- 当前进行中描述
- 关键决策记录
- 下一步计划

- [ ] **Step 2: 在 sleep-mode.md 中添加交接文档章节**

在"上下文压缩生存"和"Completion Promise"之间插入：
`## 交接文档生成（Harness Engineering — 跨 Session 连续性）`
- 触发时机：context window 即将耗尽时
- 生成内容：从 state.yaml + plan.md 组合
- 生命周期：创建 → SessionStart 读取 → 恢复后删除

---

## Task 11: 修改 Resume Protocol — 加 handoff 恢复

**Files:**
- Modify: `~/.claude/skills/autodev/steps/12-resume-protocol.md`

- [ ] **Step 1: 在"目标"之后插入 handoff 优先恢复逻辑**

插入 `## Handoff 优先恢复（Harness Engineering）` 章节：
- 优先检查 `.claude/handoff.md`（比 state.yaml 更精确）
- 存在 → 读取精确位置 → 继续执行 → 完成后删除
- 不存在 → fallback 到原有 state.yaml 恢复逻辑

---

## Task 12: 修改 helpers.md — 添加新 helper

**Files:**
- Modify: `~/.claude/skills/autodev-shared/helpers.md`

- [ ] **Step 1: 在文件末尾添加两个新 helper**

添加：
- `## #Generate-Handoff` — 从 state.yaml + plan.md 生成 .claude/handoff.md
- `## #Update-Plan-Status` — 更新 plan.md 中指定 task 的 status 和 decision

---

## Task 13: 端到端验证

- [ ] **Step 1: 验证所有新文件存在**

检查 7 个新文件全部存在：
- 3 个 hook 脚本
- harness-init SKILL.md + 2 个模板
- handoff 模板

- [ ] **Step 2: 验证所有修改后的文件包含新内容**

grep 关键词确认 8 个修改文件都包含新增内容。

- [ ] **Step 3: 验证 hook 脚本可执行**

用 `node -c` 验证语法 + 模拟输入测试不崩溃。

---

## 实施总结

| Task | 改什么 | 类型 | 文件数 |
|------|-------|------|--------|
| 1-3 | 全局 hook 脚本 | 新增 | 3 |
| 4 | settings.json 注册 hooks | 修改 | 1 |
| 5 | harness-init skill | 新增 | 3 |
| 6 | Phase 5.5 地图式 index | 修改 | 1 |
| 7 | Phase 5 加 acceptance criteria | 修改 | 1 |
| 8 | Phase 6 加 reviewer + 活文档 | 修改 | 1 |
| 9 | Phase 7 契约验收 | 修改 | 1 |
| 10 | Sleep mode handoff | 修改+新增 | 2 |
| 11 | Resume protocol handoff | 修改 | 1 |
| 12 | helpers.md 新 helper | 修改 | 1 |
| 13 | 端到端验证 | 验证 | 0 |
| **Total** | | | **15 文件** |
