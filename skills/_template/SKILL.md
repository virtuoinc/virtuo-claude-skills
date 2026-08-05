---
name: your-skill-name
description: >
  [Third person. State WHAT the skill does AND WHEN Claude should invoke it.
  Include the key action verbs and domain nouns users will say. Max 1024 chars,
  no XML tags. Example: "Processes Jira tickets and updates sprint boards. Use
  when asked to create, update, or query Jira issues, sprints, or boards."]
---

# Your Skill Name

## MCP Requirements

This skill requires the following MCP servers and tools. Each must be active in
your Claude Desktop session before this skill will work correctly.

| MCP Server   | Tool                          | Purpose                    |
|--------------|-------------------------------|----------------------------|
| `ServerName` | `ServerName:tool_name`        | Brief description           |
| `ServerName` | `ServerName:other_tool_name`  | Brief description           |

> **Remove servers that don't apply.** If a tool is only used for an optional
> step, mark it as "(optional)" in the Purpose column.

## When to use this skill

Use this skill when:
- [Trigger condition 1 — be specific about what the user says or asks]
- [Trigger condition 2]

Do NOT use this skill when:
- [Exclusion — prevents false triggers on similar requests]

## Instructions

[Keep total SKILL.md body under 500 lines. Move detailed reference material
to a separate file (e.g. REFERENCE.md) and link from here — one level deep only.]

### Step 1: [First action]

[Instructions for this step. For calls to MCP tools, specify which tool and
key parameters explicitly so Claude makes the right call every time:]

Call `ServerName:tool_name` with:
- `parameter_a`: [what value to use and where to get it]
- `parameter_b`: [what value to use and where to get it]

### Step 2: [Second action]

[Instructions. For conditional logic, use explicit decision rules:]

If [condition], call `ServerName:tool_a`.
If [other condition], call `ServerName:tool_b` instead.

### Step 3: [Verification / output]

[What Claude should confirm or return to the user when the skill completes.]

## Keywords

[comma-separated keywords that trigger this skill — match the words users
actually say, e.g.: create ticket, update issue, sprint board, Jira query]
