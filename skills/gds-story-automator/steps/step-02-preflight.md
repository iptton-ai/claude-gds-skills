---
name: 'step-02-preflight'
description: 'Select epic and stories, analyze complexity, configure execution'
nextStep: './step-03-execute.md'
---

# Step 2: Pre-flight (Epic + Story Selection)

**Goal:** Select the target epic and stories, analyze complexity, and configure the build cycle.
**Interaction mode:** Collaborative discovery and clarification.

---

## Do

### 1. Load and Parse Epics

Discover epics file from `{planning_artifacts}`:
- Search for `*epic*.md` (whole document) — use glob `{planning_artifacts}/*epic*.md`
- If not found, search for `*epic*/index.md` (sharded) — use glob `{planning_artifacts}/*epic*/index.md`
- If sharded, read index and load all epic section files

Load the discovered epics file and parse all available epics.

Display:
```
**Available Epics:**

1. Epic 1: {title} — {story_count} stories
2. Epic 2: {title} — {story_count} stories
...

Which epic? (e.g., `1`, `all`, `1,3`)
```

**Wait.**

### 2. Review Selected Epic

For the selected epic, display:
```
**Epic {N}: {title}**

Stories:
1. {story_id} — {story_title}
2. {story_id} — {story_title}
...

Total: {story_count} stories

Current sprint-status:
- {story_id}: {status}
- {story_id}: {status}
...

Which stories? (e.g., `1-3`, `all`, `1,3,5`)
```

If user hesitates, suggest `all` as default and confirm.

**Wait.**

### 3. Complexity Analysis

For each selected story, assess complexity based on:
- Number of acceptance criteria
- Technical dependencies
- Cross-story coupling
- Testing requirements

Display Complexity Matrix:
```
**Complexity Matrix:**

| Story | Title | Level | Score | Key Factors |
|-------|-------|-------|-------|-------------|
| 1-1 | {title} | Low | 3 | {factors} |
| 1-2 | {title} | Medium | 6 | {factors} |
| 1-3 | {title} | High | 9 | {factors} |

Summary: {low_count} Low, {medium_count} Medium, {high_count} High
```

### 4. Agent Configuration

Based on complexity, display the execution plan:
```
**Execution Plan:**

Stories to process: {count}
Estimated subagent invocations: {invocation_count}

For each story, the cycle is:
  create-story → dev-story → code-review → (commit)

Retrospective will trigger automatically when all stories in an epic are done.

Proceed? [Y/n]
```

**Wait.** If user says no, ask what to adjust.

### 5. Custom Instructions

```
**Any custom instructions?**

Examples:
- "Always run tests after changes"
- "Prioritize stories 3 and 5"
- "Be extra careful with database migrations"

Enter instructions or 'none':
```

If user is unsure, recommend `none` and continue.

**Wait.**

Store response as `custom_instructions` (use "" for none).

### 6. Create State Document

Create the state document at `{state_folder}/orchestration-{epic_id}-{timestamp}.md`:

```markdown
---
status: INITIALIZED
created: {timestamp}
lastUpdated: {timestamp}
epicId: {epic_id}
epicTitle: {epic_title}
storyRange: {story_ids}
currentStory: ""
currentStep: step-02-preflight
customInstructions: "{custom_instructions}"
---

# Story Automator — Epic {epic_id}: {epic_title}

## Story Progress

| Story | Create | Dev | Review | Commit | Status |
|-------|--------|-----|--------|--------|--------|
{story_rows}

<!-- Progress rows -->

## Action Log

- **[{timestamp}]** Orchestration initialized
- **[{timestamp}]** Stories selected: {story_ids}
- **[{timestamp}]** Custom instructions: {custom_instructions_or_none}
```

Display: "**State document created: {state_file_path}**"

---

## Then

→ Load, read completely, and execute `{nextStep}`
