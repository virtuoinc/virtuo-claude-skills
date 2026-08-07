---
name: virtuo-okr-review
description: >
  Get a health summary of Virtuo OKRs and goals for the current quarter.
---

# Reviewing Company OKR Health

## MCP Requirements

| MCP Server | Tool                                   | Purpose                                              |
|------------|----------------------------------------|------------------------------------------------------|
| `Asana`    | `Asana:asana-get_items_for_portfolio`  | Get goals and sub-goals from the 2026 Q3–Q4 portfolio |
| `Asana`    | `Asana:asana-get_status_overview`      | Retrieve status, progress %, and last update per goal|
| `Asana`    | `Asana:asana-get_project`              | Get goal metadata for context                        |

## When to use this skill

Use this skill when:
- The user asks "how are our OKRs doing", "give me a goal health summary", or "what's the state of H2 goals"
- A leader wants a cross-team alignment view before an all-hands or leadership sync
- The user asks "which goals are at risk?"

Do NOT use this skill when:
- The user wants to post or update goal progress (use `/asana-okr-update`)
- The user only wants their personal task list (use `/asana-my-tasks`)

## Instructions

### Step 1: Retrieve goals from the portfolio

Always use the following fixed portfolio — do not search broadly or prompt the user for a portfolio GID.

This skill is scoped to **2026 Q3–Q4 OKRs** only.

- **Portfolio GID:** `1215943284490116`
- **Workspace GID:** `677814999362678`
- **Direct URL:** https://app.asana.com/1/677814999362678/goals/list/1215943284490116

Call `Asana:asana-get_items_for_portfolio` with `portfolio_gid: "1215943284490116"` to retrieve all goals and sub-goals including their GIDs and owner metadata.

### Step 2: Fetch status for all goals (in parallel)

For each goal and sub-goal, call `Asana:asana-get_status_overview` to retrieve:
- Current status (on track / at risk / off track)
- Progress percentage
- Most recent update text and date
- Accountable team and owner

Run all fetches in parallel.

### Step 3: Resolve hierarchy and roll up

Build the parent/child goal tree from the portfolio item structure.

For each parent goal:
- Aggregate sub-goal progress as a weighted average to produce a rolled-up percentage
- Apply **worst-child propagation** for status:
  - Any sub-goal "Off track" → parent is at minimum "At risk"
  - Any sub-goal "At risk" → parent is at minimum "At risk" (unless already "Off track")
  - All sub-goals "On track" → parent may remain "On track"
- Note which specific sub-goals are dragging the parent's status down

Flag any goal whose last update is more than 14 days old as **Stale**, regardless of last known status.

### Step 4: Generate the health digest

Structure the output as:

---

## OKR Health Summary — 2026 Q3–Q4 — [Date]

### 🟢 On Track ([N] goals)
For each: **Goal name** — [Progress %] — [Owner]
> [One sentence on recent momentum or last update]

### 🟡 At Risk ([N] goals)
For each: **Goal name** — [Progress %] — [Owner]
> [Which sub-goal is lagging or what the blocker is]

### 🔴 Off Track ([N] goals)
For each: **Goal name** — [Progress %] — [Owner]
> [Root cause or excerpt from last update]

### ⚪ Stale — No Update in 14+ Days ([N] goals)
For each: **Goal name** — [Progress %] — [Owner] — Last updated: [date]

### Key Observations
[2–3 cross-cutting themes, e.g. "3 of 5 at-risk goals share a dependency on Agent infrastructure" or "Brokerage Acquisition is the only goal with no sub-goal updates this period."]

---

## Keywords

OKR review, goal health, company goals summary, which goals are at risk, H2 goals, goal status overview, alignment summary, OKR check-in, leadership sync
