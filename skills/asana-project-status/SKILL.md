---
name: asana-project-status
description: >
  Generates a structured project status report in Asana by analyzing task
  completion, overdue items, blockers, and upcoming milestones, then posts a
  confirmed status update to the project. Surfaces % complete, wins, and risks
  in a concise narrative the user reviews before publishing. Invoke with
  /asana-project-status.
---

# Project Status Updates

## MCP Requirements

| MCP Server | Tool                                       | Purpose                                              |
|------------|--------------------------------------------|------------------------------------------------------|
| `Asana`    | `Asana:asana-get_projects`                 | List projects when none is specified                 |
| `Asana`    | `Asana:asana-search_objects`               | Locate a project by name                             |
| `Asana`    | `Asana:asana-get_project`                  | Fetch project metadata and completion data           |
| `Asana`    | `Asana:asana-search_tasks`                 | Get open, overdue, and recently completed tasks      |
| `Asana`    | `Asana:asana-get_task`                     | Get full details on key overdue or milestone tasks   |
| `Asana`    | `Asana:asana-get_status_overview`          | Retrieve the last posted status for comparison       |
| `Asana`    | `Asana:asana-create_project_status_update` | Post the confirmed status update                     |

## When to use this skill

Use this skill when:
- The user asks "post a status update for [project]", "what's the status of [project]", or "write a project update"
- A project owner wants to run their weekly status reporting workflow
- The user asks "which projects need a status update?"

Do NOT use this skill when:
- The user wants to update progress on a company OKR/goal (use `/asana-okr-update`)
- The user only wants to browse their personal task list (use `/asana-my-tasks`)

## Instructions

### Step 1: Identify the project

If the user named a specific project, call `Asana:asana-search_objects` to locate it.

If no project is named, call `Asana:asana-get_projects` to list the user's recent projects and ask which one to update.

### Step 2: Fetch project data (in parallel)

Call in parallel:
- `Asana:asana-get_project` — metadata: name, owner, due date, percent complete, section structure
- `Asana:asana-get_status_overview` — previous status update for before/after comparison
- `Asana:asana-search_tasks` filtered to the project with `completed: false` — all open tasks
- `Asana:asana-search_tasks` filtered to the project with `completed: true` and a completion date within the last 7 days — recent wins

### Step 3: Analyze task data

From open tasks, identify:
- **Overdue**: tasks with a due date before today
- **Blocked**: tasks tagged as blocked, or with no assignee and no due date past 7 days
- **Upcoming milestones**: tasks due within 14 days marked as milestones or high priority

Calculate:
- Overall % complete (completed tasks / total tasks)
- Count of overdue tasks and count of tasks due this week

For overdue tasks and upcoming milestones that need more detail, call `Asana:asana-get_task` in parallel to get their description, assignee, and dependencies.

Suggest status color:
- **At risk**: overdue task count > 15% of total tasks, OR project due date within 14 days with < 50% complete
- **Off track**: project due date within 7 days with < 40% complete, OR more than 3 blocked tasks with no resolution
- **On track**: everything else

### Step 4: Draft the status narrative

Compose a 3–5 sentence status narrative:
1. Overall progress (% complete, one key milestone achieved this period)
2. What's going well (top recent completions)
3. What's at risk or blocked (specific tasks with owner names where available)
4. Next milestone and its target date

### Step 5: Present to user and confirm

Show the user:
- Computed stats: % complete, overdue count, tasks due this week, recent completions
- Suggested status color with one-line reasoning
- Draft narrative

Ask the user to confirm or edit the status color and narrative before posting.

### Step 6: Post the confirmed update

Call `Asana:asana-create_project_status_update` with:
- `parent`: the project GID
- `status_type`: `on_track` / `at_risk` / `off_track`
- `title`: "Status Update — [current date]"
- `text`: the confirmed narrative

Confirm: "Status update posted to [Project Name]."

## Keywords

project status, post project update, weekly status, project health, project report, what's the status of, project update, status report
