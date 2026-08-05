---
name: asana-my-tasks
description: >
  Retrieves the user's open Asana tasks, groups them by urgency and project,
  and cross-references today's calendar to surface tasks relevant to upcoming
  meetings. Presents a prioritized triage view and applies any re-prioritizations
  or due-date changes the user confirms. Invoke with /asana-my-tasks.
---

# Personal Task Triage

## MCP Requirements

| MCP Server      | Tool                                        | Purpose                                                        |
|-----------------|---------------------------------------------|----------------------------------------------------------------|
| `Asana`         | `Asana:asana-get_my_tasks`                  | Fetch all open tasks assigned to the user                      |
| `Asana`         | `Asana:asana-get_task`                      | Retrieve full details for each task                            |
| `Asana`         | `Asana:asana-update_tasks`                  | Apply priority or due-date changes                             |
| `Asana`         | `Asana:asana-save_task_changes_confirm`     | Confirm task updates before saving                             |
| `Microsoft 365` | `Microsoft 365:get_calendar_events`         | Fetch today's meetings for contextual prioritization (optional)|

## When to use this skill

Use this skill when:
- The user asks "what are my tasks", "what do I need to do today", or "triage my tasks"
- The user wants to review and reprioritize their open work in Asana
- The user asks "what's overdue" or "what's due this week"

Do NOT use this skill when:
- The user wants to create new tasks (use `/asana-task-capture`)
- The user wants to review company OKRs or goal progress (use `/asana-okr-review`)

## Instructions

### Step 1: Fetch tasks and calendar in parallel

Call `Asana:asana-get_my_tasks` to retrieve all open tasks assigned to the current user.

Simultaneously (if Microsoft 365 is available), call `Microsoft 365:get_calendar_events` with:
- `startDateTime`: start of today
- `endDateTime`: end of today
- `top`: 10

### Step 2: Enrich task details

For each task returned, call `Asana:asana-get_task` in parallel batches to retrieve:
- Due date
- Project(s)
- Priority / custom fields
- Description / notes

### Step 3: Group and prioritize

Organize tasks into urgency groups:

1. **Overdue** — due date is before today
2. **Due today** — due date is today
3. **Due this week** — due within the next 7 days
4. **Upcoming / No due date** — due beyond 7 days or unscheduled

Within each group, surface tasks related to today's meetings at the top by matching the task's project name or title against meeting subjects and attendee names from the calendar.

### Step 4: Present the triage view

Display the grouped task list. For each task show: task name, project, due date, and a one-line description if present.

Then ask: "Are there any tasks to reprioritize, reschedule, or mark complete?"

### Step 5: Apply confirmed changes

For each change the user confirms, call `Asana:asana-update_tasks` with:
- `task_id`: the task's GID
- Updated fields (due date, priority, assignee, etc.)

Then call `Asana:asana-save_task_changes_confirm` to commit the changes.

Confirm each update to the user: "[Task name] — [what changed]."

## Keywords

my tasks, task triage, what do I need to do, what's overdue, prioritize tasks, due today, task list, Asana tasks, daily tasks
