---
description: Sync selected user stories or tasks to Azure DevOps for the given feature using ado-remote-mcp server
---

## User Input

```text
$ARGUMENTS
```

## Outline

## Make sure ado-remote-mcp server is running.

**CRITICAL WORKFLOW - Follow these steps IN ORDER:**

This command syncs user stories from spec.md OR tasks from tasks.md for the give feature to Azure DevOps as work items using ado-remote-mcp server.

### Step 1: Collect Azure DevOps Configuration (ASK USER IN CHAT FIRST)

**DO THIS BEFORE ANYTHING ELSE**: Ask the user for these configuration details **in the chat**:

1. **Check for saved configuration** first:
   - Look for `~/.speckit/ado-config.json` (Windows: `C:\Users\<username>\.speckit\ado-config.json`)
   - If file exists, read and display the saved values

2. **If configuration exists**, ask user:

   ```text
   I found your saved Azure DevOps configuration:
   - Organization: <saved-org>
   - Project: <saved-project>
   - Area Path: <saved-area-path>
   
   Would you like to use these settings? (yes/no)
   ```

3. **If no configuration OR user says no**, ask these questions **ONE BY ONE** in chat:

   ```text
   What is your Azure DevOps Organization name?
   (e.g., "MSFTDEVICES" from https://dev.azure.com/MSFTDEVICES)
   ```

   **Wait for response, then ask:**

   ```text
   What is your Azure DevOps Project name?
   (e.g., "Devices")
   ```

   **Wait for response, then ask:**

   ```text
   What is your Area Path?
   (e.g., "Devices\SW\ASPX\CE\Portals and Services")
   ```
   
4. **Wait for response and then Store the responses** as variables for later use

### Step 2: Locate feature folder and ask user to select feature for which story/task needs to be created.
1. List out the feature directories present in the root folder. The feature folder name will be in the format: <N> - <FeatureName>
   E.g.: 001-Dose-Report-Sharing

2. **Display found features in chat** like this:
   ```text
   Found 3 features in root folder:
   [1] Dose Report Sharing
   [2] Review Procedure Creation
   [3] Acquisition Procedure Creation
   ```
 
3. Ask user to select any one feature option for which we need to create user stories/tasks.
   ```text
   Your selection:
   ```
   **Wait for response, then continue:**

### Step 3: Ask user to select any one option from below:
   ```text
   Please select one option from below:
   [1] Sync only user stories
   [2] Synch tasks and link to user stories
   ```
   **Wait for response, then continue:**

### Step 4a: User Stories selection

   **If user selected 'Sync only user stories' option:**
   1. Look for the `spec.md` file in the user selected feature directory
   
   2. Read `spec.md` and extract all user stories using pattern:

   ```markdown
   ### User Story <N> - <Title> (Priority: P<N>)
   ```
   3. **Display found stories in chat** like this:

   ```text
   Found 5 user stories in spec.md belongs to <Feature-Folder>:
   
   [1] User Story 1 - User Authentication (P1)
   [2] User Story 2 - Profile Management (P1)
   [3] User Story 3 - Password Reset (P2)
   [4] User Story 4 - Session Management (P2)
   [5] User Story 5 - Account Deletion (P3)
   ```
   4. User story options selection:
   **Ask user in chat**:
   ```text
   Which user stories would you like to sync to Azure DevOps?

   Options:
   - all - Sync all user stories for that feature
   - 1,2,3 - Sync specific stories (comma-separated)
   - 1-5 - Sync a range of stories

   Your selection:
   ```
   **Wait for user response**, then parse selection:
   - "all" -> select all stories
   - "1,3,5" -> select stories 1, 3, and 5
   - "1-5" -> select stories 1 through 5
   - Empty/invalid -> prompt again

   5. Show Confirmation
   **After getting selection, show what will be created**:

   ```text
   You selected X user stories to sync:

     - US1 - User Story 1 - <Story Title>
     - US2 - User Story 2 - <Story Title>
  
   Is this correct? (yes/no)
   ```
   *** After getting the confirmation only execute next steps ***

   6. Prepare User Stories Work Items

   For each selected user story, prepare user story work item data:

   ```javascript
   {
	 type: "Story",
	 title: `User Story ${storyNumber} - ${storyTitle}`,
	 fields: {
	   "System.Description": `${description}\n\n**Why this priority**: ${whyPriority}\n\n**Independent Test**: ${independentTest}`,
	   "Microsoft.VSTS.Common.AcceptanceCriteria": formatAcceptanceCriteria(scenarios),
	   "Microsoft.VSTS.Common.Priority": convertPriority(priority), // P1->1, P2->2, P3->3
	   "System.Tags": `spec-kit; ${featureName}; user-story`,
	   "System.AreaPath": areaPath || `${project}`,
	   "System.IterationPath": `${project}` // Can be enhanced to detect current sprint
	 }
   }
   ```

   **Acceptance Criteria Formatting**:

   ```text
   Scenario 1:
   Given: <given>
   When: <when>
   Then: <then>

   Scenario 2:
   Given: <given>
   When: <when>
   Then: <then>
   ```
   7. Use ado-remote-mcp MCP tools to check and create the user stories in the ADS
   Before creating a new story, first check whether the user story **already exists** in ADS.
   To check the story already exists or not, search all the WorkItemType=Story under **AreaPath having the same story title and sme tags**
   If the **user story already exists** in ADS, then overwrite it instead of creating a new one. 
   **Create a new user story only when it does not exists in ADS**
   Use **ado-remote-mcp always** to create the user story with the above data in ADS

   8. Display ado-remote-mcp progress and results

   **Real-time feedback**: Display status as each work item is created:**

   ```text
   Creating User Story 1 of 3...
   [OK] Created User Story 1: Display Success Notifications (#12345)
   
   Creating User Story 2 of 3...
   [OK] Created User Story 2: Edit Notifications (#12346)
   
   Creating User Story 3 of 3...
   [FAIL] Failed User Story 3: Delete Notifications (Permission denied)
   ```

### Step 4b: Tasks selection [Synch tasks and link to user stories]

   **If user selected 'Synch tasks and link to user stories' option:**
   1. Look for the `tasks.md` file in the user selected feature directory
   
   2. Read `tasks.md` and extract all tasks and its partent user stories using pattern:

   ```markdown
   - [ ] T001 [P] [US1] Task description
   ```
   3. **Display found stories in chat** like this:
   ```text
   Found 25 tasks in tasks.md:
   
   User Story 1 (8 tasks):
     [1] T001 - Setup authentication service
     [2] T002 - Create login endpoint
     [3] T003 - Implement password validation
     ...
   
   User Story 2 (12 tasks):
     [8] T010 - Design user profile schema
     [9] T011 - Create profile API
     ...
   
   No parent (5 unlinked tasks):
     [20] T050 - Update documentation
     ...
   ```
   4. Tasks options selection:
   **Ask user in chat**:
   ```text
   Which tasks would you like to sync to Azure DevOps?

   You can select by:
   - all - Sync all tasks
   - us1 - All tasks for User Story 1
   - us1,us2 - All tasks for multiple User Stories
   - 1,2,3 - Specific task numbers (comma-separated)
   - 1-10 - Range of task numbers

   Your selection:
   ```
   **Wait for user response**, then parse selection:

   - "all" -> select all tasks
   - "us1" -> select all tasks linked to User Story 1
   - "us1,us3" -> select all tasks linked to User Story 1 and 3
   - "1,3,5" -> select tasks 1, 3, and 5
   - "1-10" -> select tasks 1 through 10
   - Empty/invalid -> prompt again
   
   5. Show Confirmation
   **After getting selection, show what will be created**:
   ```text
   You selected X tasks to sync:

   User Story 1 (3 tasks):
     - T001 - Setup authentication service
     - T002 - Create login endpoint
     - T003 - Implement password validation

   User Story 2 (2 tasks):
     - T005 - Design user profile schema
     - T006 - Create profile API

   Is this correct? (yes/no)
   ```
   ***After getting the confirmation only execute next steps***

   6. Prepare Task Work Items
   For each new task, assign first 512 characters from task description and then add '...' at the end and assign it to task's title.
   Before creating a new task, first check whether the task **alreay exists** in ADS.
   To check the task already exists or not, search all the WorkItemType=Task under **AreaPath having the same task's title and sme tags**
   If the **task already exists** in ADS, then overwrite it instead of creating new one. **Create new task only when it does not exists in ADS**
   
   ```javascript
   {
	 type: "Task",
	 title: `T${taskNumber} - ${description}`,
	 fields: {
	   "System.Description": `${description}`,
	   "System.Tags": `spec-kit; ${featureName}; task`,
	   "System.AreaPath": areaPath || `${project}`,
	   "System.IterationPath": `${project}` // Can be enhanced to detect current sprint
	 }
   }
   ```
   7. Use ado-remote-mcp MCP tools to check and create the tasks and link the user stories to tasks in the ADS
   
   8. Display ado-remote-mcp progress and results

   **Real-time feedback**: Display status as each work item is created:**

   ```text
   Creating Task 1 of 30...
   Linked Task 1 to User Story 1
   [OK] Created Task 1: Display Success Notifications (#12345)
   
   Creating Task 2 of 30...
   Linked Task 2 to User Story 2
   [OK] Created Task 2: Edit Notifications (#12346)
   
   Creating Task 3 of 30...
   No parent user story exists. Parent is not mapped.
   [FAIL] Failed Task 3: Delete Notifications (Permission denied)
   ```   
   
### Step 5: Display overall results summary

   **Display Results**
   Show summary table:

   ```markdown
   ## [SUCCESS] Azure DevOps Sync Complete

   **Organization**: MSFTDEVICES  
   **Project**: Devices  
   **Feature**: photo-album-management  
   **Synced**: 3 of 4 user stories

   ### Created Work Items

   | Story | Title | Priority | Work Item | Status |
   |-------|-------|----------|-----------|--------|
   | 1 | Create Photo Albums | P1 | [#12345](https://dev.azure.com/MSFTDEVICES/Devices/_workitems/edit/12345) | [OK] Created |
   | 2 | Add Photos to Albums | P1 | [#12346](https://dev.azure.com/MSFTDEVICES/Devices/_workitems/edit/12346) | [OK] Created |
   | 3 | Delete Albums | P2 | [#12347](https://dev.azure.com/MSFTDEVICES/Devices/_workitems/edit/12347) | [OK] Created |

   ### View in Azure DevOps

   - **Boards**: [https://dev.azure.com/MSFTDEVICES/Devices/_boards](https://dev.azure.com/MSFTDEVICES/Devices/_boards)
   - **Work Items**: [https://dev.azure.com/MSFTDEVICES/Devices/_workitems](https://dev.azure.com/MSFTDEVICES/Devices/_workitems)
   - **Backlog**: [https://dev.azure.com/MSFTDEVICES/Devices/_backlogs/backlog](https://dev.azure.com/MSFTDEVICES/Devices/_backlogs/backlog)

## Key Rules

- Use ado-remote-mcp MCP server for ADS access
- Save org/project/area to `~/.speckit/ado-config.json` for reuse
- Title format: User Stories = "User Story {#} - {title}", Tasks = "T{#} - {desc}"
- Priority mapping: P1->1, P2->2, P3->3, P4->4
- Auto-link tasks to parent user stories via `[US#]` references
- Continue on failure, report all errors at end
