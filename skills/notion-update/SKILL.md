---
name: notion-update
description: >
  Updates an existing Notion page by appending new content or modifying its
  title and properties. Resolves the target page by URL or title search, reads
  the current content to preserve context, then applies the requested changes.
  Useful for adding meeting notes to a running doc, updating a status section,
  or correcting a page title. Invoke with /notion-update.
---

# Notion Update

## MCP Requirements

| MCP Server | Tool                         | Purpose                                         |
|------------|------------------------------|-------------------------------------------------|
| `notion`   | `notion:notion-search`       | Find the target page by title                   |
| `notion`   | `notion:notion-fetch`        | Read current content before writing             |
| `notion`   | `notion:notion-update-page`  | Apply content or property changes               |

## When to use this skill

Use this skill when:
- The user wants to add notes, a section, or new content to an existing page
- The user wants to update the title or status of a Notion page
- The user asks to "append", "add to", "update", or "edit" a Notion page

Do NOT use this skill when:
- The user wants to create a brand new page (use `/notion-draft`)
- The user only wants to read or summarize the page (use `/notion-summarize`)

## Instructions

### Step 1: Resolve the target page
- **URL provided**: proceed to Step 2.
- **Title provided**: call `notion:notion-search` with the title. Confirm with
  the user if the match is ambiguous.

### Step 2: Read current content
Call `notion:notion-fetch` on the resolved page to understand the existing
structure. This ensures new content is appended in the right style and that
existing content is not accidentally overwritten.

### Step 3: Clarify the change
If the user has not specified exactly what to add or change, ask one focused
question: *"What would you like to add or update on this page?"*

Determine the type of change:
- **Append** — add new content at the bottom or after a named section
- **Title/property update** — change the page title or a property value
- **Section rewrite** — replace a specific named section's content

### Step 4: Apply the update
Call `notion:notion-update-page` with the target page ID and the new content
or property values. Structure appended content consistently with the existing
page style (headings, bullets, etc.).

### Step 5: Confirm
Return the updated page title and direct link. Briefly describe what was
changed.

## Keywords
update notion page, add to notion, append notion, edit notion doc, add notes to notion, update status notion
