---
name: asana-task-capture
description: >
  Extracts action items from freeform user-supplied text (pasted emails, Slack
  threads, conversation notes, or dictated lists) and creates them as Asana
  tasks. Previews each extracted task with suggested assignee, due date, and
  project before confirming creation. For tasks from meeting transcripts, use
  /meeting-recap instead. Invoke with /asana-task-capture.
---

# Capturing Tasks from Freeform Input

## MCP Requirements

| MCP Server | Tool                                   | Purpose                                             |
|------------|----------------------------------------|-----------------------------------------------------|
| `Asana`    | `Asana:asana-search_objects`           | Find the right project to assign tasks to           |
| `Asana`    | `Asana:asana-get_projects`             | List available projects when no project hint exists |
| `Asana`    | `Asana:asana-get_users`                | Resolve assignee names to Asana user IDs            |
| `Asana`    | `Asana:asana-create_task_preview_v4`   | Preview extracted tasks before creating             |
| `Asana`    | `Asana:asana-create_task_confirm`      | Confirm and create each task                        |

## When to use this skill

Use this skill when:
- The user pastes an email, Slack message, conversation excerpt, or dictated list and asks to "capture tasks", "create tasks from this", or "add these as todos"
- The user dictates a list of action items to add directly to Asana
- The user forwards a block of text and says "make tasks out of this"

Do NOT use this skill when:
- Tasks are being extracted from a meeting transcript — use `/meeting-recap`, which handles transcript retrieval, full recap email, and task creation end-to-end
- The user wants to review existing tasks (use `/asana-my-tasks`)

## Instructions

### Step 1: Receive the input

If the user has not already pasted text, prompt: "Paste the email, message, or notes you'd like to extract tasks from."

Accept any freeform text. Do not attempt to retrieve transcripts or calendar data — this skill processes user-supplied text only.

### Step 2: Extract action items

Analyze the text and extract each discrete action item. For each, identify:
- **Task name**: a concise, action-verb description (e.g. "Send proposal to Acme by Friday")
- **Assignee**: name or email if mentioned; otherwise leave blank
- **Due date**: explicit date or relative reference (e.g. "by Friday", "next week") — convert to ISO date (YYYY-MM-DD) where inferable; leave blank if not stated
- **Project hint**: any project, team, or initiative name mentioned in context

Show the extracted list to the user and ask for corrections before proceeding.

### Step 3: Resolve project and assignees (in parallel)

For each task:

**Project**: If a project hint was identified, call `Asana:asana-search_objects` with the hint as the query to find the best-match Asana project. If no hint or no match found, call `Asana:asana-get_projects` and prompt the user to select a project.

**Assignee**: If an assignee name was identified, call `Asana:asana-get_users` to resolve it to an Asana user ID. If no match is found, create the task unassigned and note it in the summary.

### Step 4: Preview and confirm

For each task, call `Asana:asana-create_task_preview_v4` with:
- `name`: task name
- `assignee`: resolved user ID (if found)
- `due_on`: ISO date (if found)
- `projects`: resolved project GID (if found)
- `notes`: the source excerpt that generated this task

Show the previews to the user. Allow any edits before confirming.

### Step 5: Create confirmed tasks

For each task the user approves, call `Asana:asana-create_task_confirm` to finalize creation.

Report back: "Created [N] tasks in Asana:" then list each task name and project.

## Keywords

capture tasks, create tasks from text, add action items, tasks from email, tasks from Slack, dictate tasks, extract action items, add todos
