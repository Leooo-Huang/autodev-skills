# AutoDev — Spec-Driven AI Development Framework

A Claude Code skill system for structured, document-driven software development. Combines **pipeline orchestration** with **BMAD micro-file architecture** to keep AI agents focused, constrained, and verifiable.

## Philosophy

```
Design docs (soft constraint) -> Index + Rules (refined constraint) -> Hooks (hard constraint)
```

**Specifications are the source of truth. Code is an expression of the spec.** Unlike pure spec-driven approaches, AutoDev acknowledges that docs drift from code, so it includes **drift detection, GAN-style adversarial review, and self-correction** at every phase.

## Architecture

**Pipeline backbone** (8 phases + 3 GAN checkpoints) + **BMAD file organization** (4-layer separation per skill):

```
SKILL.md (entry, ~60-90 lines) -> steps/ (JIT-loaded micro-files) -> templates/ -> checklists/
```

Each skill loads only the current step into context, reducing token waste by ~70%.

## Skills

### Main Pipeline (`/autodev`)

| Phase | Skill | Output |
|-------|-------|--------|
| 1. Ideation | `autodev-ideation` | `*-ideation.md` |
| 2. Design | `superpowers:brainstorming` + `/last30days` OSS scan | `*-oss-scan.md` + `*-design.md` |
| 3. UI/UX | `autodev-ui` | `*-ui.md` |
| 4. API | `autodev-api` | `*-api.md` |
| 5. Planning | `autodev-plan` | `*-plan.md` (with contract-style acceptance criteria) |
| 5.5 Index | `autodev-compress` | `*-index.md` + `*-rules.md` |
| 5.9 **Plan GAN** | `autodev-review --target plan` | `plan_review_history` |
| 6. Development | `superpowers:subagent-driven-development` + per-task **Code GAN** + **UI GAN** | code commits |
| 7. Verification | `autodev-verify` (contract + redlines + static + runtime + acceptance) | verification report |
| 7.5 **Global GAN** | `autodev-review --target global` | `docs/pipeline/global-review.md` |
| 8. Delivery | `superpowers:finishing-a-development-branch` | merge / PR |

### Building Blocks (also independently usable)

| Skill | Command | Purpose |
|-------|---------|---------|
| `autodev-ideation` | `/autodev-ideation` | Domain-expert product ideation from raw user need |
| `autodev-brainstorm` | `/autodev-brainstorm` | Diverge -> evaluate -> converge on a design (post-research) |
| `autodev-ui` | `/autodev-ui` | UI/UX design — IA, user flows, page structure, L3/L4 visual + motion specs |
| `autodev-api` | `/autodev-api` | API design — data model, endpoints, contracts |
| `autodev-plan` | `/autodev-plan` | Generate a contract-style implementation plan (`acceptance_criteria` + `status` per task) with downgrade-signal scanning |
| `autodev-compress` | `/autodev-compress` | Compress design docs into `index.md` (map) + `rules.md` (coding rules) for development-phase context efficiency |
| `autodev-review` | `/autodev-review` | GAN-style adversarial review — spawn an independent skeptical reviewer, score on 4 dimensions, iterate or pivot. Targets: `plan` / `code` / `ui` / `global` |
| `autodev-verify` | `/autodev-verify` | Unified verification — contract acceptance + redline scan + static check + runtime validation + `acceptance-testing` |

### Auxiliary Workflows

| Skill | Command | Purpose |
|-------|---------|---------|
| `autodev-add` | `/autodev-add` | Add a new feature to an existing project (incremental design + implementation, full-depth ideation, code-reuse scanning) |
| `autodev-iterate` | `/autodev-iterate` | Bug fix / improvement on existing features — 7-phase: research -> design -> implement -> verify, with smart dispatch |
| `autodev-sync` | `/autodev-sync` | Sync doc completion status + design drift detection (5 dimensions, INFO/WARN/ERROR severity) |
| `autodev-reverse` | `/autodev-reverse` | Generate AutoDev-format design docs from an existing codebase (incl. L3/L4 UI extraction) |
| `autodev-polish` | `/autodev-polish` | GAN-style visual polishing — Generator-Evaluator loop to lift UI to a target design system (Stripe / Linear / Vercel etc.) |

### Shared Resources

| File | Purpose |
|------|---------|
| `autodev-shared/helpers.md` | Shared operation snippets (read docs, update state, stub scan, etc.) |
| `autodev-shared/checklists/quality-redlines.md` | 6 quality redlines in checkbox format |
| `autodev-shared/templates/` | Drift-report and handoff templates |

### External Skills Used by the Pipeline

| Skill | Role |
|-------|------|
| `acceptance-testing` | Final E2E acceptance verification (called by `autodev-verify`) |
| `skill-improver` | Self-evolving skill repair when a skill misbehaves |

## Quality Assurance (Unique to AutoDev)

Features not found in BMAD, Spec Kit, Kiro, or OpenSpec:

- **6 Quality Redlines**: no placeholders, no mocks, no downgrade, correct versions, OSS reuse over self-build, no emoji as UI icons
- **Stub Code Detection**: 5 signal patterns (hardcoded empty values, TODO comments, mock data, uncalled APIs, short-circuit conditions)
- **Design Drift Detection**: 5 dimensions (tech stack, API endpoints, data models, page routes, core dependencies) with INFO/WARN/ERROR severity
- **3 GAN Checkpoints** ([Anthropic Harness Design](https://www.anthropic.com/engineering/harness-design-long-running-apps)):
  - **Plan GAN** (Phase 5.9) — adversarial review of the plan before any code is written
  - **Code + UI GAN** (Phase 6, per task) — adversarial review of each implemented task
  - **Global GAN** (Phase 7.5) — adversarial review of the whole project after verification
- **Contract-Style Plans**: every task in `plan.md` carries `acceptance_criteria` and `status` fields, enforced by `autodev-verify`
- **OSS-First Design**: Phase 2 mandates `/last30days` + WebSearch scan of every R-capability; reuse is default, self-build needs justification
- **Design Self-Correction**: when implementation deviates from design docs, `autodev-iterate` proactively repairs the docs
- **Tech Freshness Verification**: WebSearch validation of library versions during design phase

## Stats

| Metric | Count |
|--------|-------|
| Core AutoDev skills | 14 |
| External skills bundled | 2 (`acceptance-testing`, `skill-improver`) |
| Total SKILL.md lines | ~1,300 |
| Step / template / checklist files | ~110 |

## Installation

```bash
# Clone
gh repo clone Leooo-Huang/autodev-skills
# or: git clone https://github.com/Leooo-Huang/autodev-skills.git

# Copy to Claude Code skills directory (Linux/macOS)
cp -r autodev-skills/autodev*       ~/.claude/skills/
cp -r autodev-skills/acceptance-testing ~/.claude/skills/
cp -r autodev-skills/skill-improver     ~/.claude/skills/
```

```powershell
# Windows (PowerShell)
Copy-Item -Recurse autodev-skills\autodev*           $HOME\.claude\skills\
Copy-Item -Recurse autodev-skills\acceptance-testing $HOME\.claude\skills\
Copy-Item -Recurse autodev-skills\skill-improver     $HOME\.claude\skills\
```

## Usage

| Goal | Command |
|------|---------|
| Build a new project end-to-end | `/autodev "your project idea"` |
| Add a feature to an existing project | `/autodev-add "new feature"` |
| Fix a bug or iterate on existing code | `/autodev-iterate "what to change"` |
| Generate AutoDev docs from existing code | `/autodev-reverse` |
| Sync doc completion status | `/autodev-sync` |
| Polish UI visuals to a design system | `/autodev-polish` |
| Run only one phase | `/autodev-ideation` `/autodev-ui` `/autodev-api` `/autodev-plan` `/autodev-compress` `/autodev-review` `/autodev-verify` |

## Comparison with Other Frameworks

| Dimension | AutoDev | BMAD | Spec Kit | Kiro |
|-----------|---------|------|---------|------|
| Complexity | Medium (14 skills, JIT loading) | High (21 agents) | Low (4 files) | Medium |
| Quality gates | 6 redlines + stub detection + drift detection + 3 GAN checkpoints | Adversarial review | Constitution + checklist | EARS notation |
| Iteration support | `autodev-iterate` (7-phase) + `autodev-add` (incremental) | bmad-correct-course | Manual | Manual |
| Doc sync | `autodev-sync` (auto, 5-dim drift) | None | Manual | None |
| Token efficiency | JIT micro-file loading (~70% reduction) | High consumption | Moderate | Moderate |
| Visual quality loop | `autodev-polish` (GAN) | None | None | None |

## License

MIT
