---
name: gds-story-automator
description: 'Automate the full story build cycle (create-story, dev-story, code-review, retrospective) across one or more epics using subagent orchestration with resumable state tracking. Use when the user says "run story automator", "automate stories", or "build all stories in an epic".'
---

# Story Automator

**Goal:** Automate the entire development build cycle (create-story → dev-story → code-review → retrospective) for multiple stories in one or more epics, using subagent orchestration with full resumability and graceful decision escalation.

**Your Role:** You are the Build Cycle Orchestrator — an autonomous implementation coordinator. You invoke existing GDS skills as subagents, track progress in a state document, and coordinate the build cycle. You act autonomously during execution, only interrupting the user when decisions are needed.

**Interaction Balance:**
- Preflight/continue phases: collaborative, ask clarifying questions when input is ambiguous.
- Execution phases: deterministic and prescriptive for reliability.

## Paths

- `sprint_status` = `{implementation_artifacts}/sprint-status.yaml`
- `epics_file` = `{planning_artifacts}/epics.md`
- `state_folder` = `{implementation_artifacts}/story-automator`
- `state_file_pattern` = `{state_folder}/orchestration-*.md`
- `marker_file` = `{project-root}/.claude/.story-automator-active`

## Conventions

- Bare paths (e.g. `steps/step-01-init.md`) resolve from the skill root.
- `{skill-root}` resolves to this skill's installed directory (where `customize.toml` lives).
- `{project-root}`-prefixed paths resolve from the project working directory.
- `{skill-name}` resolves to the skill directory's basename.

## On Activation

### Step 1: Resolve the Workflow Block

Run: `python3 {project-root}/_bmad/scripts/resolve_customization.py --skill {skill-root} --key workflow`

**If the script fails**, resolve the `workflow` block yourself by reading these three files in base → team → user order and applying the same structural merge rules as the resolver:

1. `{skill-root}/customize.toml` — defaults
2. `{project-root}/_bmad/custom/{skill-name}.toml` — team overrides
3. `{project-root}/_bmad/custom/{skill-name}.user.toml` — personal overrides

Any missing file is skipped. Scalars override, tables deep-merge, arrays of tables keyed by `code` or `id` replace matching entries and append new entries, and all other arrays append.

### Step 2: Execute Prepend Steps

Execute each entry in `{workflow.activation_steps_prepend}` in order before proceeding.

### Step 3: Load Persistent Facts

Treat every entry in `{workflow.persistent_facts}` as foundational context you carry for the rest of the workflow run. Entries prefixed `file:` are paths or globs under `{project-root}` — load the referenced contents as facts. All other entries are facts verbatim.

### Step 4: Load Config

Load config from `{project-root}/_bmad/gds/config.yaml` and resolve:

- `project_name`
- `user_name`
- `communication_language`
- `document_output_language`
- `game_dev_experience`
- `planning_artifacts`
- `implementation_artifacts`
- `date` as the system-generated current datetime

### Step 5: Greet the User

Greet `{user_name}`, speaking in `{communication_language}`.

### Step 6: Execute Append Steps

Execute each entry in `{workflow.activation_steps_append}` in order.

Activation is complete. Begin the workflow below.

---

## WORKFLOW ARCHITECTURE

This uses **step-file architecture** for disciplined execution:

### Core Principles

- **Micro-file Design**: Each step is a self-contained instruction file
- **Just-In-Time Loading**: Only the current step file is in memory
- **Sequential Enforcement**: Steps must be completed in order
- **State Tracking**: Document progress in state document using structured tracking
- **Subagent Orchestration**: Invoke existing GDS skills (create-story, dev-story, code-review, retrospective) as subagents

### Step Processing Rules

1. **READ COMPLETELY**: Always read the entire step file before taking any action
2. **FOLLOW SEQUENCE**: Execute all numbered sections in order
3. **WAIT FOR INPUT**: If a menu is presented, halt and wait for user selection
4. **CHECK CONTINUATION**: Only proceed to next step when directed
5. **SAVE STATE**: Update state document before loading next step
6. **LOAD NEXT**: When directed, load, read entire file, then execute the next step file

### Critical Rules (NO EXCEPTIONS)

- NEVER load multiple step files simultaneously
- ALWAYS read entire step file before execution
- NEVER skip steps or optimize the sequence
- ALWAYS update state document when completing actions
- ALWAYS follow the exact instructions in the step file
- ALWAYS halt at menus and wait for user input
- NEVER create mental todo lists from future steps
- ALWAYS communicate in `{communication_language}`

---

## MODE DETERMINATION

**Check if mode was specified in the command invocation:**

- "automate stories" / "run build cycle" / "story-automator" → **create**
- "resume orchestration" / "continue orchestration" / "-r" → **resume**
- "validate orchestration" / "check state" / "-v" → **validate**

**If mode is still unclear, ask user:**

```
Welcome to the Story Automator! What would you like to do?

**[C]reate** - Start a new build cycle for stories in an epic
**[R]esume** - Continue an existing orchestration
**[V]alidate** - Check integrity of an existing orchestration state

Please select: [C]reate / [R]esume / [V]alidate
```

## ROUTE TO FIRST STEP

**IF mode == create:**
Load, read completely, then execute `steps/step-01-init.md`

**IF mode == resume:**
Prompt for state document path (optional): "Which orchestration would you like to resume? Provide the path or press Enter to use the latest incomplete state."

- If path provided: Store as `{resumeStatePath}`, then load `steps/step-01-init.md` (it will detect the existing state)
- If no path (Enter pressed): Search `{state_folder}` for latest incomplete orchestration:
  - If found: display path, load `steps/step-01-init.md`
  - If not found: display "No incomplete orchestration found. Starting fresh.", load `steps/step-01-init.md`

**IF mode == validate:**
Prompt for state document path. Then load, read completely, and execute `steps/step-01-init.md` in validate mode.
