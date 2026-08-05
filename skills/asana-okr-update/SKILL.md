---
name: asana-okr-update
description: >
  Updates progress and status on Asana company goals (OKRs). Fetches linked
  projects and tasks to auto-calculate a suggested progress percentage, handles
  parent/child goal hierarchy by rolling sub-goal progress up to parent goals,
  and posts a status update with a narrative summary after user confirmation.
  Invoke with /asana-okr-update.
---

# Updating OKR / Goal Progress

## MCP Requirements

| MCP Server | Tool                                       | Purpose                                            |
|------------|--------------------------------------------|----------------------------------------------------|
| `Asana`    | `Asana:asana-search_objects`               | Find goals by name or owner                        |
| `Asana`    | `Asana:asana-get_status_overview`          | Retrieve current status and progress for a goal    |
| `Asana`    | `Asana:asana-get_items_for_portfolio`      | Get projects and sub-goals linked to a goal        |
| `Asana`    | `Asana:asana-get_project`                  | Fetch project completion data for progress calc    |
| `Asana`    | `Asana:asana-search_tasks`                 | Get task-level completion data within a project    |
| `Asana`    | `Asana:asana-create_project_status_update` | Post the confirmed status update to the goal       |

## When to use this skill

Use this skill when:
- The user says "update my OKRs", "post goal progress", or "update [goal name]"
- The user wants to record a weekly or milestone check-in on a company goal
- The user asks "what should I update for [time period]?"

Do NOT use this skill when:
- The user wants a read-only health summary of all goals (use `/asana-okr-review`)
- The user wants project-level (not goal-level) status updates (use `/asana-project-status`)

## Instructions

### Step 1: Identify goals to update

If the user named a specific goal, call `Asana:asana-search_objects` with the goal name as the query.

If no goal is specified, call `Asana:asana-search_objects` filtered to goals owned by the current user. Present the list and ask which goals to update. Default to updating all owned goals if the user says "all my goals."

### Step 2: Retrieve current status and linked data (in parallel)

For each goal, call in parallel:
- `Asana:asana-get_status_overview` — last posted status, current progress %, and time period
- `Asana:asana-get_items_for_portfolio` — all linked projects and sub-goals

### Step 3: Calculate progress

**For leaf goals (no sub-goals):**

For each linked project, call `Asana:asana-get_project` to get the total and completed task counts. If more granular data is needed, call `Asana:asana-search_tasks` filtered to the project to count completed vs. total tasks.

Calculate a weighted average completion percentage across all linked projects.

**For parent goals (with sub-goals):**

Recursively apply Steps 2–3 for each sub-goal first. Then roll up:
- Suggested parent progress = weighted average of sub-goal progress values
- Parent status = downgrade to "At risk" if any sub-goal is "At risk" or "Off track"

### Step 4: Present proposed update to user

For each goal, show:
- **Current progress**: last recorded %
- **Calculated progress**: computed % with source (N linked projects / sub-goals)
- **Suggested status**: On track / At risk / Off track — with one-line reasoning
- **Draft narrative**: 2–4 sentences covering what moved, what's blocked, and what's next

Ask the user to:
1. Confirm or override the progress percentage
2. Confirm or adjust the status color
3. Confirm or edit the narrative

### Step 5: Post updates

Post sub-goal updates before parent goals so the parent narrative can reference sub-goal outcomes.

For each confirmed goal, call `Asana:asana-create_project_status_update` with:
- `parent`: the goal GID
- `status_type`: `on_track` / `at_risk` / `off_track`
- `title`: "Progress Update — [current date]"
- `text`: the confirmed narrative
- `progress`: confirmed percentage

Confirm each posted update: "Updated [Goal Name] — [status] at [progress]%."

## Keywords

update OKRs, goal progress, post update, check in on goals, weekly OKR, goal status, H2 goals, company goals progress, OKR check-in
