---
description: Create a new ClickUp task
argument-hint: [description]
---

# /task:new $ARGUMENTS

Create a new ClickUp task through an interview flow.

## Workflow

1. **Interview user** using AskUserQuestion tool:

   **Question 1: Task Type**
   - Feature (new functionality)
   - Bug (something broken)
   - Refactor (code improvement)
   - Chore (maintenance, config)

   **Question 2: Brief Description**
   - If $ARGUMENTS provided, use as starting point
   - Otherwise ask: "What should this task accomplish?"

   **Question 3: Acceptance Criteria**
   - Ask: "What are the acceptance criteria? (bullet points)"

2. **Format task description** using standard template:
   ```markdown
   ## Description
   {user description}

   ## Acceptance Criteria
   - [ ] {criterion 1}
   - [ ] {criterion 2}
   ...
   ```

3. **Create task via clickup-task-agent**:
   ```
   Use Task tool with subagent_type="task:clickup-task-agent":

   "Create ClickUp task:
   - Name: {task_name}
   - Description: {formatted_description}
   - Type tag: {feature|bug|refactor|chore}

   Return: task ID and URL"
   ```

4. **Ask user**: "Start work on this task now?"
   - Yes: Run `/task:start CU-{new_id}`
   - No: Complete, show task URL

## Notes

- Task is created in the default list (configured in ClickUp workspace)
- Tags are applied based on task type
- User can immediately start working or save for later
