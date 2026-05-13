---
name: 'step-01-init'
description: 'Initialize orchestrator, verify prerequisites, check for existing state, install stop hook'
nextStep: './step-02-preflight.md'
---

# Step 1: Initialize

**Goal:** Verify prerequisites, check for existing orchestration state, install stop hook, and prepare for build cycle.

---

## Do

### 1. Check Sprint Status (MANDATORY)

Verify that `{sprint_status}` exists.

**IF file does NOT exist:**
```
**Sprint status file not found.**

Expected: `{implementation_artifacts}/sprint-status.yaml`

This file is required before running the story automator.
Please run the **sprint-planning** workflow first to generate it.
```
**HALT** — Do not proceed.

**IF file exists:** Store path for later reference. Continue.

### 2. Install Stop Hook (MANDATORY)

The Stop hook prevents the orchestrator from being interrupted mid-workflow.

**Check if hook already installed:**
- Read `{project-root}/.claude/settings.json`
- Look for a Stop hook command containing `story-automator-active`

**IF NOT installed, add the hook:**

Read the current `{project-root}/.claude/settings.json`, merge the following Stop hook into the `hooks.Stop` array, and write back:

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "test -f {project-root}/.claude/.story-automator-active && echo '{\"decision\":\"block\",\"reason\":\"Story automator is in progress. Use /clear to override if needed.\"}' || echo '{\"decision\":\"allow\"}'",
            "timeout": 5
          }
        ]
      }
    ]
  }
}
```

**IF newly installed:**
Display:
```
**Stop Hook Installed**

I've added the story-automator Stop hook to .claude/settings.json.
This prevents the orchestrator from randomly stopping mid-workflow.

Please restart this Claude session for the hook to take effect.
After restarting, run the story-automator workflow again.
```
**HALT** — Do not proceed until user restarts.

**IF already installed:** Display: "✓ Stop hook verified". Continue.

### 3. Check for Existing Orchestration State

Search `{state_folder}` for `orchestration-*.md` files.

**IF found (latest incomplete):**
- Display: "**Found existing orchestration in progress.**"
- Show: epic name, current story, current step, last updated
- Ask: "Resume this orchestration? [Y/n]"

  - If Y: Load the state document, extract `currentStory` and `currentStep`, then load and execute the appropriate step file.
  - If n: Continue to step 4 (start fresh).

**IF none found:** Continue to step 4.

### 4. Welcome

Display:
```
**Welcome to Story Automator (GDS).**

I'll automate story implementation by invoking GDS skills as subagents
(create-story → dev-story → code-review), handling code review loops,
and triggering retrospectives when epics complete.

Everything is logged in a state document for full resumability.
```

### 5. Ensure Output Folder

Create `{state_folder}` if it doesn't exist.

### 6. Create Marker File

Write a JSON marker file to `{marker_file}`:

```json
{
  "startedAt": "{timestamp}",
  "heartbeat": "{timestamp}",
  "status": "initializing"
}
```

This marker:
- Signals that the story automator is active
- Is checked by the Stop hook to block premature interruption
- Contains a heartbeat timestamp — if older than 30 minutes, the hook allows stop (assumes orchestrator crashed)
- Is removed in step-04-wrapup when orchestration completes

---

## Then

→ Load, read completely, and execute `{nextStep}`
