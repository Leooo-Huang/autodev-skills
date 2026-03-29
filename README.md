# AutoDev — Spec-Driven AI Development Framework

A Claude Code skill system for structured, document-driven software development. Combines **pipeline orchestration** with **BMAD micro-file architecture** to keep AI agents focused and constrained.

## Philosophy

```
Design docs (soft constraint) -> Index + Rules (refined constraint) -> Hooks (hard constraint)
```

**Specifications are the source of truth. Code is an expression of the spec.** But unlike pure spec-driven approaches, AutoDev acknowledges that docs drift from code, so it includes built-in **drift detection and self-correction**.

## Architecture

**Pipeline backbone** (8 phases, sequential) + **BMAD file organization** (4-layer separation per skill):

```
SKILL.md (entry, ~60-85 lines) -> steps/ (JIT-loaded micro-files) -> templates/ -> checklists/
```

Each skill loads only the current step into context, reducing token waste by ~70%.

## Skills

### Main Pipeline (`/autodev`)

| Phase | Skill | Output |
|-------|-------|--------|
| 1. Ideation | `autodev-ideation` | `*-ideation.md` |
| 2. Design | `autodev-brainstorm` | `*-design.md` |
| 3. UI/UX | `autodev-ui` | `*-ui.md` |
| 4. API | `autodev-api` | `*-api.md` |
| 5. Planning | built-in | `*-plan.md` |
| 5.5 Index | built-in | `*-index.md` + `*-rules.md` |
| 6. Development | subagents | code commits |
| 7. Verification | `acceptance-testing` | tests pass |
| 8. Delivery | built-in | PR/merge + hooks config |

### Auxiliary

| Skill | Command | Purpose |
|-------|---------|---------|
| `autodev-add` | `/autodev-add` | Add a new feature to an existing project (incremental design + implementation) |
| `autodev-iterate` | `/autodev-iterate` | Bug fix / improvement on existing features (7-phase: research -> design -> implement -> verify) |
| `autodev-sync` | `/autodev-sync` | Sync doc completion status + design drift detection |
| `autodev-reverse` | `/autodev-reverse` | Generate AutoDev-format docs from existing codebase |

### Shared Resources

| File | Purpose |
|------|---------|
| `autodev-shared/helpers.md` | 5 shared operation snippets (read docs, update state, stub scan, etc.) |
| `autodev-shared/checklists/quality-redlines.md` | 4 quality redlines in checkbox format |
| `autodev-shared/templates/design-drift-report.md` | Drift report template |

## Quality Assurance (Unique to AutoDev)

Features not found in BMAD, Spec Kit, Kiro, or OpenSpec:

- **4 Quality Redlines**: No placeholders, no mocks, no downgrade, correct versions
- **Stub Code Detection**: 5 signal patterns (hardcoded empty values, TODO comments, mock data, uncalled APIs, short-circuit conditions)
- **Design Drift Detection**: 5 dimensions (tech stack, API endpoints, data models, page routes, core dependencies) with INFO/WARN/ERROR severity
- **Design Self-Correction**: When implementation deviates from design docs, autodev-iterate proactively fixes the docs
- **Tech Freshness Verification**: WebSearch validation of library versions during design phase

## Stats

| Metric | Count |
|--------|-------|
| Skills | 10 + 2 auxiliary (acceptance-testing, skill-improver) |
| SKILL.md total lines | ~650 (avg ~60/skill) |
| Step files | 83 |
| Template files | 9 |
| Checklist files | 12 |
| Total files | 107 |

## Installation

Copy the skill directories to your Claude Code skills folder:

```bash
# Clone
git clone https://github.com/Leooo-Huang/autodev-skills.git

# Copy to Claude Code skills directory
cp -r autodev-skills/autodev* ~/.claude/skills/
cp -r autodev-skills/acceptance-testing ~/.claude/skills/
cp -r autodev-skills/skill-improver ~/.claude/skills/
```

Then use `/autodev "your project idea"` to start a full pipeline, or `/autodev-add "new feature"` to add features incrementally.

## Comparison with Other Frameworks

| Dimension | AutoDev | BMAD (39.5k stars) | Spec Kit (74.8k stars) | Kiro |
|-----------|---------|---------------------|----------------------|------|
| Complexity | Medium | High (21 agents) | Low (4 files) | Medium |
| Quality gates | 4 redlines + stub detection + drift detection | Adversarial review | Constitution + checklist | EARS notation |
| Iteration support | autodev-iterate (7 phases) + autodev-add | bmad-correct-course | Manual | Manual |
| Doc sync | autodev-sync (auto) | None | Manual | None |
| Token efficiency | JIT micro-file loading (~70% reduction) | High consumption | Moderate | Moderate |

## License

MIT
