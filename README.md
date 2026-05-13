# Claude GDS Skills

**inspired by claud [terryso's bmad skills](https://github.com/terryso/claude-bmad-skills)**

Claude Code Skills collection for GDS (Game Development Sprint) methodology(in BMAD).

[中文文档](README_CN.md)

## Installation

### Option 1: Claude Code Plugin Marketplace (Recommended)

```bash
/plugin marketplace add yltech/claude-gds-skills
/plugin install gds-skills
```

### Option 2: npx skills

```bash
npx skills add yltech/claude-gds-skills
```

### Option 3: Git Submodule

```bash
git submodule add https://github.com/yltech/claude-gds-skills.git .agents/gds-skills
```

### Option 4: Clone & Copy

```bash
git clone https://github.com/yltech/claude-gds-skills.git
cp -r claude-gds-skills/skills/* ~/.claude/skills/
```

## Available Skills

### Skill Overview

| Skill | Execution Mode | Pipeline | Isolation |
|-------|---------------|----------|-----------|
| gds-story-pipeline | Subagent | Configurable | None |
| gds-story-pipeline-worktree | Subagent | Configurable | Worktree |
| gds-epic-pipeline-worktree | Batch | Configurable | Worktree |

> **Recommended**: Use `gds-story-pipeline` or `gds-story-pipeline-worktree` for configurable workflow.

---

### gds-story-pipeline

Run configurable GDS pipeline for Godot game story delivery using subagent.

```bash
/gds-story-pipeline 1-1
# Or omit argument to auto-select
/gds-story-pipeline
```

**Features:**
- Configurable workflow via `references/workflow-steps.md`
- Each step runs in isolated subagent (fresh context)
- Customizable pipeline steps

**Default pipeline:**
1. Create user story
2. Generate GDD tests (scene structure, node hierarchy, physics validation)
3. Development (GDScript, scenes, assets)
4. Code review (Godot-specific: signals, performance, scene design)
5. Trace test coverage

---

### gds-story-pipeline-worktree

Run configurable GDS pipeline in isolated worktree, merge only after tests pass.

```bash
/gds-story-pipeline-worktree 1-1
# Or omit argument to auto-select
/gds-story-pipeline-worktree
```

**Features:**
- All features of `gds-story-pipeline`
- Plus worktree isolation
- Conditional merge (tests pass + no HIGH/MEDIUM issues)

**Difference from gds-story-pipeline:**
| Feature | pipeline | pipeline-worktree |
|---------|----------|-------------------|
| Working method | Current branch | Isolated worktree |
| Code isolation | None | Complete isolation |
| Merge condition | None enforced | Tests pass + fixes complete |
| Safety level | Medium | High |

---

### gds-epic-pipeline-worktree

Deliver entire Epic using configurable pipeline in isolated worktrees.

```bash
/gds-epic-pipeline-worktree 3
# Or omit argument to auto-select
/gds-epic-pipeline-worktree
```

**Features:**
- Uses `gds-story-pipeline-worktree` for each story
- Configurable pipeline via workflow-steps.md
- Worktree isolation per story

**Execution logic:**
1. Collect all incomplete stories under Epic
2. Sort by Story number ascending
3. Call `/gds-story-pipeline-worktree` for each story sequentially
4. Next story starts only after previous completes
5. If any fails, stop and preserve state

**Use cases:**
- Batch deliver entire Epic
- Automate multi-story sequential development
- Ensure each story passes tests independently

---

## Directory Structure

```
claude-gds-skills/
├── README.md
├── README_CN.md
├── LICENSE
├── .claude-plugin/
│   ├── marketplace.json
│   └── plugin.json
└── skills/
    ├── gds-story-pipeline/
    │   ├── SKILL.md
    │   └── references/workflow-steps.md
    ├── gds-story-pipeline-worktree/
    │   ├── SKILL.md
    │   └── references/workflow-steps.md
    └── gds-epic-pipeline-worktree/
        └── SKILL.md
```

## Choosing the Right Skill

**For single story:**
- **Recommended**: `gds-story-pipeline` or `gds-story-pipeline-worktree` (configurable workflow)
- Need worktree isolation? -> `gds-story-pipeline-worktree`

**For entire Epic:**
- **Recommended**: `gds-epic-pipeline-worktree` (configurable workflow)

## GDS vs BMAD

| Aspect | BMAD | GDS |
|--------|------|-----|
| Target | General software | Godot game development |
| Test type | ATDD (acceptance) | GDD (game design document) |
| Review focus | Code quality | Scene structure, GDScript, signals, performance |
| Asset handling | N/A | Sprites, audio, fonts, shaders, level data |
| Physics validation | N/A | Gravity, pendulum, collision, bounce |

## Contributing

Issues and Pull Requests welcome to add new GDS skills.

## License

MIT
