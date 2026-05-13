---
name: 'step-04-wrapup'
description: 'Finalize orchestration — summary, learnings, cleanup (terminal step)'
---

# Step 4: Wrap-up

**Goal:** Generate summary, capture learnings, finalize state.
**Interaction mode:** Structured wrap-up with recommendation output.

---

## Do

### 1. Load Final State

From the state document, extract:
- Story progress table
- Action log
- Stories completed vs total
- Code review cycles
- Escalations encountered

### 2. Generate Summary

Display:
```
**Story Automator — Build Cycle Complete**

**Epic:** {epic_title}
**Stories Processed:** {completed}/{total}
**Deferred (plateau):** {deferred_count}
**Code Review Cycles:** {total_review_cycles}
**Escalations:** {escalation_count}
**Retrospectives Triggered:** {retro_count}

**Story Results:**
| Story | Status | Review Cycles | Notes |
|-------|--------|---------------|-------|
{results_table}

**Deferred Stories:**
{deferred_list_or_none}

**Action Log Summary:**
{action_log_summary}
```

### 3. Capture Learnings

Analyze the run for patterns:
- Common code review issues
- Steps needing escalation
- What worked well
- Areas for improvement

If `{state_folder}/learnings.md` exists: load and append new entry.
Else: create new file with first entry.

Learnings entry format:
```markdown
## Run: {date} — Epic {epic_id}

**Stories:** {count}
**Successes:** {success_patterns}
**Issues:** {issue_patterns}
**Recommendations:** {recommendations}
```

### 4. Recommendations

Based on patterns observed, present actionable suggestions:
- If common review issues: suggest preventive measures
- If escalations occurred: suggest process improvements
- If high complexity stories: suggest breaking down further next time
- If deferred stories: suggest manual review or alternative approach for plateau stories

### 5. Finalize State

Update state document:
- Set `status = COMPLETE`
- Set `completedAt = {timestamp}`
- Append final summary to Action Log

### 6. Remove Marker File

Delete `{marker_file}` to signal orchestration is complete.

### 7. Workflow Complete

Display:
```
**Story Automator workflow complete!**

All stories have been processed through the build cycle.
Retrospectives were triggered automatically when each epic completed.

State document: {state_file_path}
Learnings: {state_folder}/learnings.md
```

---

## End

**Workflow terminates here.**
