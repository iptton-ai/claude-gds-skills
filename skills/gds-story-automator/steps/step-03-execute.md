---
name: 'step-03-execute'
description: 'Main execution loop — create, dev, review, commit for each story with verification, plateau detection, and retry intelligence'
nextStep: './step-04-wrapup.md'
---

# Step 3: Execute Build Cycle

**Goal:** Autonomously execute all stories through the full build cycle. Escalate only when decisions are needed.
**Interaction mode:** Deterministic autonomous execution.

---

## Setup

Load from state document:
- `storyRange`, `currentStory`, `currentStep`
- `customInstructions`

**IF resuming** (`currentStory` is set): Skip to that story in the loop.
**IF fresh**: Display "**Starting build cycle for {count} stories...**"

Initialize tracking variables:
- `plateau_tracker = {}` — maps `{story_id}:{step}` to consecutive failure count
- `failure_log = {}` — maps `{story_id}:{step}` to list of failure reasons

---

## Story Loop

**FOR EACH story in range:**

Update state document:
- Set `currentStory = {story_id}`
- Set `currentStep = step-03-execute`
- Update `lastUpdated` and `heartbeat` in marker file
- Append to Action Log: "Starting story {story_id}"

Display: "**Story {N}/{total}: {title}**"

---

### A. Create Story

*Skip if story file already exists in `{implementation_artifacts}/{story_id}.md`*

**Invoke create-story subagent:**

Use the Agent tool to spawn a subagent with the `gds-create-story` skill:

```
Agent prompt: "Use the gds-create-story skill to create story {story_id} from epic {epic_num}.
The epic file is at {epics_file}.
Implementation artifacts: {implementation_artifacts}
Planning artifacts: {planning_artifacts}
Sprint status: {sprint_status}
Communication language: {communication_language}
Document output language: {document_output_language}
Game dev experience: {game_dev_experience}

{custom_instructions_if_any}

Create the story file and update sprint-status.yaml to mark it as ready-for-dev."
```

**Post-Execution Verification (MANDATORY — do NOT trust subagent self-report):**

After subagent returns, independently verify:
1. Story file exists at `{implementation_artifacts}/{story_id}.md`
2. Read `{sprint_status}` — check `development_status[{story_id}]` equals `ready-for-dev`

```
IF story file exists AND sprint-status shows ready-for-dev:
  → SUCCESS, proceed to B

IF story file exists BUT sprint-status not updated:
  → Log warning: "Story file created but sprint-status not updated"
  → Manually update sprint-status[{story_id}] = "ready-for-dev"
  → SUCCESS, proceed to B

IF story file does NOT exist:
  → FAILURE, enter retry logic
```

**Retry Logic (up to 3 attempts with plateau detection):**

```
FOR attempt = 1 to 3:
  IF attempt > 1:
    # Enhanced retry: include failure context
    Append to prompt: "PREVIOUS ATTEMPT FAILED: {failure_reason}. Address this specific issue."
    Log: "[{timestamp}] create-story attempt {attempt}/3 for {story_id}: retrying after '{failure_reason}'"

  Invoke subagent
  Run post-execution verification

  IF verified:
    BREAK → success

  IF not verified:
    Record failure_reason in failure_log[{story_id}:create]
    Increment plateau_tracker[{story_id}:create]

    # Plateau detection: same step fails 3 times in a row
    IF plateau_tracker[{story_id}:create] >= 3:
      Log: "[{timestamp}] PLATEAU: create-story for {story_id} failed 3 times consecutively"
      Display: "**Plateau detected** for create-story {story_id}. Deferring to next story."
      Mark story as "deferred" in state document
      GOTO next story

    IF attempt == 3 AND not verified:
      ESCALATE to user:
      - Show all 3 failure reasons
      - Options: [1] Retry manually  [2] Skip story  [3] Abort orchestration
      WAIT for user choice
```

**On success:**
- Update Story Progress: mark create-story as `done`
- Append to Action Log: "Story {story_id}: create-story done"
- Display: `[story {N}/{total}] create-story -> done`

---

### B. Dev Story

**Invoke dev-story subagent:**

Use the Agent tool to spawn a subagent with the `gds-dev-story` skill:

```
Agent prompt: "Use the gds-dev-story skill to implement story {story_id}.
The story file is at {implementation_artifacts}/{story_id}.md
Implementation artifacts: {implementation_artifacts}
Sprint status: {sprint_status}
Communication language: {communication_language}
Document output language: {document_output_language}
Game dev experience: {game_dev_experience}

{custom_instructions_if_any}

Implement ALL tasks in the story file. Follow TDD (red-green-refactor).
Mark story status to 'review' when complete."
```

**Post-Execution Verification (MANDATORY):**

After subagent returns, independently verify:
1. Read story file — check Status section shows `review` or `done`
2. Read `{sprint_status}` — check `development_status[{story_id}]` equals `review`

```
IF story status is review AND sprint-status shows review:
  → SUCCESS, proceed to C

IF story status is review BUT sprint-status not updated:
  → Log warning, manually update sprint-status
  → SUCCESS, proceed to C

IF story tasks are incomplete (unchecked [ ] remain):
  → FAILURE — subagent stopped early
  → Enter retry logic with specific incomplete task info
```

**Retry Logic (up to 3 attempts with plateau detection):**

Same pattern as Create Story, with enhanced failure context:
- Include which tasks are still incomplete
- Include any error messages from dev-story subagent
- Plateau detection: if same task fails 3 times → DEFER story

**On success:**
- Update Story Progress: mark dev-story as `done`
- Append to Action Log: "Story {story_id}: dev-story done"
- Display: `[story {N}/{total}] dev-story -> done`

---

### C. Code Review

**Invoke code-review subagent:**

Use the Agent tool to spawn a subagent with the `gds-code-review` skill:

```
Agent prompt: "Use the gds-code-review skill to review the implementation of story {story_id}.
The story file is at {implementation_artifacts}/{story_id}.md
Implementation artifacts: {implementation_artifacts}
Communication language: {communication_language}

Review the code changes adversarially. Report findings with severity levels (CRITICAL, HIGH, MEDIUM, LOW)."
```

**Review Loop (up to 5 cycles with sprint-status verification):**

```
reviewCycle = 1
maxCycles = 5

WHILE reviewCycle <= maxCycles:
  Invoke code-review subagent

  # Determine review result
  IF review result is "approve" OR 0 CRITICAL issues:
    Log: "Code review passed (cycle {reviewCycle})"
    BREAK → proceed to D

  IF review result is "changes requested" AND has CRITICAL issues:
    # Re-invoke dev-story with review feedback
    Invoke dev-story subagent with prompt including:
      "Address these CRITICAL review findings: {findings_list}"

    # Verify dev fixed the issues (post-execution verification)
    Read story file — check tasks are complete

    reviewCycle++

  IF 5 cycles without approval:
    ESCALATE to user with summary:
    - Total review cycles
    - Remaining CRITICAL/HIGH issues per cycle
    - Options: [1] Manual fix  [2] Re-run with different agent  [3] Skip story
    WAIT for user choice
```

**Sprint-Status Verification After Review (MANDATORY):**

After review loop exits as "approve":
1. Read `{sprint_status}` — check `development_status[{story_id}]` equals `done`
2. If not "done", check story file status

```
IF sprint-status shows "done":
  → SUCCESS

IF sprint-status shows "review" (reviewer didn't update):
  → Log warning: "Code review passed but sprint-status not updated to done"
  → Manually update sprint-status[{story_id}] = "done"
  → SUCCESS

IF sprint-status shows other:
  → FAILURE — investigate and escalate
```

**On success:**
- Update Story Progress: mark code-review as `done`
- Append to Action Log: "Story {story_id}: code-review done (took {N} cycles)"
- Display: `[story {N}/{total}] code-review -> done`

---

### D. Git Commit

Commit the story implementation:

```bash
cd {project-root}
git add -A
git commit -m "feat: implement story {story_id} - {story_title}"
```

**On success:**
- Update Story Progress: mark commit as `done`
- Append to Action Log: "Story {story_id}: committed"
- Display: `[story {N}/{total}] commit -> done`

**On failure:** Log warning, escalate if critical. Story is still marked as review-done.

---

### E. Story Complete

Display: "**Story {N} complete: {story_id} — {title}**"

Update state document:
- Mark entire story row as `done`
- Update `lastUpdated`
- Update `heartbeat` in marker file
- Clear plateau_tracker entries for this story (successful completion resets counters)

---

### F. Check Epic Completion

After each story completes, check if ALL stories in the current epic are done:

1. Read `{sprint_status}`
2. Find all stories for this epic (keys matching `{epic_num}-*`)
3. Check if all are marked "done"

**IF all stories in epic are done:**

Display: "**Epic {epic_number} complete! All stories done. Triggering retrospective...**"

Invoke retrospective subagent:

```
Agent prompt: "Use the gds-retrospective skill to run a retrospective for epic {epic_number}.
Implementation artifacts: {implementation_artifacts}
Planning artifacts: {planning_artifacts}
Sprint status: {sprint_status}
Communication language: {communication_language}
Document output language: {document_output_language}
Game dev experience: {game_dev_experience}
User name: {user_name}

Run the full retrospective workflow."
```

**Retrospective rules:**
- Non-blocking: if retrospective fails, log warning and continue
- Never escalates: failures are safely skipped
- Appends to Action Log: "Epic {epic_number}: retrospective {completed|skipped}"

**IF epic not complete:** Continue to next story.

---

**END FOR EACH**

After all stories complete, update state:
- Set `currentStep = step-04-wrapup`
- Set `status = EXECUTION_COMPLETE`

Display: "**All {count} stories completed! Proceeding to wrap-up...**"

---

## Then

→ Load, read completely, and execute `{nextStep}`
