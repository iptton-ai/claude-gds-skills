# GDS Story Pipeline Workflow Steps

## Pipeline Configuration

Execute the following steps in order using Task tool.
Each step runs with "yolo" mode for auto-approval.

### Step 1: Create User Story
- Command: `/gds-create-story {STORY_ID} yolo`
- Description: Creates story file with context from GDD and level design docs
- Return: Story ID, Title, Created files

### Step 2: Generate GDD Tests
- Command: `/gds-testarch-gdd {STORY_ID} yolo`
- Description: Generate failing game design tests (GDD red phase) - validates Godot scene structure, node hierarchy, physics parameters, and gameplay mechanics against design spec
- Return: GDD checklist and test files

### Step 3: Development
- Command: `/gds-dev-story {STORY_ID} yolo`
- Description: Implement story in Godot - create/modify scenes, scripts (GDScript), assets, and configurations (TDD green phase)
- Return: Modified files, Changes summary

### Step 4: Code Review
- Command: `/gds-code-review {STORY_ID} yolo`
- Description: Adversarial code review for Godot-specific issues - scene structure, GDScript quality, performance concerns, signal wiring
- Return: Conclusion (pass/needs-fix), Issues by severity

### Step 5: Trace Test Coverage
- Command: `/gds-testarch-trace {STORY_ID} yolo`
- Description: Generate traceability matrix and quality gate decision - maps story requirements to Godot scenes, scripts, and test coverage
- Return: Coverage percentage, Gate decision

## Post-Pipeline

After all steps complete:
1. Update sprint-status.yaml: story status -> done
2. Update story document: Status: done, Tasks: done

## Customization

To modify the pipeline:
- Add/remove steps in this file
- Change step commands
- Reorder steps (update step numbers)
