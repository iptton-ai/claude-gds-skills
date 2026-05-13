---
name: gds-story-pipeline
description: Run configurable GDS pipeline for Godot game story delivery using subagent
argument-hint: <story-number> e.g., 1-1 or 2-3 (optional, auto-selects if omitted)
---

# GDS Story Pipeline

Complete the delivery pipeline for story `{ARGUMENT}` using configurable Godot game development workflow.

## Pre-step: Determine Story Number

If `{ARGUMENT}` is empty or not provided:

1. Read `_gds-output/implementation-artifacts/sprint-status.yaml` (or `docs/sprint/sprint-status.yaml`) to find stories
2. Find the first story with status "todo" or "in-progress"
3. Use that story number as `{STORY_ID}`
4. If no story found, ask user to specify story number

The story number format is typically `X-Y` (e.g., `1-1`, `2-3`).

## Execution Strategy

1. Read workflow steps from **references/workflow-steps.md**
2. Execute each step sequentially using Task tool (general-purpose agent)
3. Output progress after each step
4. After pipeline, update status to done

## Workflow Steps

Read and execute steps from **references/workflow-steps.md**.

For each step defined there, you MUST use the **Task tool** to execute in a subagent:

```
Task(
  subagent_type: "general-purpose",
  description: "<Step description>",
  prompt: "Execute the command: <COMMAND_WITH_STORY_ID>
Return: 1) Step completion status 2) Key outputs 3) Any issues to note"
)
```

### Execution Flow

For each step:
1. Replace `{STORY_ID}` with the actual story number in the prompt
2. Call Task tool with the step's description and prompt
3. Wait for completion and capture result
4. Output progress: `[X/5] Step Name - Status`
5. If step fails, stop and report error

## Progress Display

After each step, output progress:

```
Pipeline Progress: [X/5] ████████░░░░ 40%

Step X: <Step Name>
   Result: <Brief result summary>
```

## Error Handling

If any step fails:
1. Stop executing subsequent steps
2. Output error information:
   ```
   Pipeline Failed at Step X: <Step Name>

   Error: <Error details>

   Suggested actions:
   - Check the story file for issues
   - Run the failed step manually: <command>
   - Fix the issue and restart pipeline
   ```
3. Do NOT proceed to next steps

## Post-Pipeline: Update Status

After ALL steps complete successfully:

1. **Update sprint-status.yaml**:
   - Find the story entry
   - Change status from `in-progress` to `done`

2. **Update story document** (if exists):
   - Change `Status:` to `done`
   - Mark all tasks with done

Output final summary:
```
Pipeline Complete!

Story: {STORY_ID}
Status: done

Steps completed: 5/5
- Create User Story
- Generate GDD Tests
- Development
- Code Review
- Trace Test Coverage
```

## Configuration

To customize the pipeline workflow, edit:
**references/workflow-steps.md**

Changes supported:
- Add/remove steps
- Modify step commands
- Reorder steps
- Change descriptions
