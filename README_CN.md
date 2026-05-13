# Claude GDS Skills

GDS (Godot Development Sprint) 方法论的 Claude Code Skills 集合。

**致敬 [Nick 的 bmad skill](https://github.com/terryso/claude-bmad-skills)**

[English](README.md)

## 安装

### 方式 1: Claude Code 插件市场（推荐）

```bash
/plugin marketplace add yltech/claude-gds-skills
/plugin install gds-skills
```

### 方式 2: npx skills

```bash
npx skills add yltech/claude-gds-skills
```

### 方式 3: Git Submodule

```bash
git submodule add https://github.com/yltech/claude-gds-skills.git .agents/gds-skills
```

### 方式 4: Clone & Copy

```bash
git clone https://github.com/yltech/claude-gds-skills.git
cp -r claude-gds-skills/skills/* ~/.claude/skills/
```

## 可用 Skills

### 技能总览

| 技能 | 执行模式 | 管道 | 隔离 |
|------|---------|------|------|
| gds-story-pipeline | 子代理 | 可配置 | 无 |
| gds-story-pipeline-worktree | 子代理 | 可配置 | Worktree |
| gds-epic-pipeline-worktree | 批量 | 可配置 | Worktree |

> **推荐**：使用 `gds-story-pipeline` 或 `gds-story-pipeline-worktree`，支持可配置工作流。

---

### gds-story-pipeline

使用子代理运行可配置的 GDS 管道交付 Godot 游戏用户故事。

```bash
/gds-story-pipeline 1-1
# 或不传参数，自动选择
/gds-story-pipeline
```

**特性：**
- 通过 `references/workflow-steps.md` 配置工作流
- 每个步骤在独立子代理中运行（全新上下文）
- 可自定义管道步骤

**默认管道：**
1. 创建用户故事
2. 生成 GDD 测试（场景结构、节点层级、物理参数验证）
3. 开发实现（GDScript、场景、资源）
4. 代码审查（Godot 特定：信号、性能、场景设计）
5. 追踪测试覆盖

---

### gds-story-pipeline-worktree

在独立 worktree 中运行可配置的 GDS 管道，测试通过后才合并。

```bash
/gds-story-pipeline-worktree 1-1
# 或不传参数，自动选择
/gds-story-pipeline-worktree
```

**特性：**
- 包含 `gds-story-pipeline` 的所有特性
- 额外的 worktree 隔离
- 条件合并（测试通过 + 无 HIGH/MEDIUM 问题）

**与 gds-story-pipeline 的区别：**
| 特性 | pipeline | pipeline-worktree |
|------|----------|-------------------|
| 工作方式 | 当前分支 | 独立 worktree |
| 代码隔离 | 无 | 完全隔离 |
| 合并条件 | 无强制要求 | 测试通过才合并 |
| 安全性 | 中 | 高 |

---

### gds-epic-pipeline-worktree

使用可配置管道在独立 worktree 中交付整个 Epic。

```bash
/gds-epic-pipeline-worktree 3
# 或不传参数，自动选择
/gds-epic-pipeline-worktree
```

**特性：**
- 使用 `gds-story-pipeline-worktree` 处理每个故事
- 通过 workflow-steps.md 配置管道
- 每个故事独立 worktree 隔离

**执行逻辑：**
1. 收集 Epic 下所有未完成的故事
2. 按 Story 编号升序排序
3. 逐个调用 `/gds-story-pipeline-worktree` 交付
4. 前一个故事完成才开始下一个
5. 任一失败则停止，保留状态

**适用场景：**
- 批量交付整个 Epic
- 自动化多故事顺序开发
- 确保每个故事独立测试通过

---

## 目录结构

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

## 如何选择合适的 Skill

**单个故事交付：**
- **推荐**：`gds-story-pipeline` 或 `gds-story-pipeline-worktree`（可配置工作流）
- 需要 worktree 隔离？ -> `gds-story-pipeline-worktree`

**整个 Epic 交付：**
- **推荐**：`gds-epic-pipeline-worktree`（可配置工作流）

## GDS 与 BMAD 的区别

| 方面 | BMAD | GDS |
|------|------|-----|
| 目标 | 通用软件开发 | Godot 游戏开发 |
| 测试类型 | ATDD（验收测试） | GDD（游戏设计文档验证） |
| 审查重点 | 代码质量 | 场景结构、GDScript、信号、性能 |
| 资源处理 | 无 | 精灵图、音频、字体、Shader、关卡数据 |
| 物理验证 | 无 | 重力、钟摆运动、碰撞、弹力 |

## 贡献

欢迎提交 Issue 和 Pull Request 添加新的 GDS skills。

## License

MIT
