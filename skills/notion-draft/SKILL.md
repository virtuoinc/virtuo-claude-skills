---
name: notion-draft
description: >
  Creates a new structured Notion page from a user description, brief, or set
  of raw notes. Resolves a named parent page via search if no page ID is
  supplied, then creates the page with formatted headings, bullets, and
  sections. Department plugins can specialize this skill by pre-filling a
  default parent page. Invoke with /notion-draft.
---

# Notion Draft

## MCP Requirements

| MCP Server | Tool                       | Purpose                                       |
|------------|----------------------------|-----------------------------------------------|
| `notion`   | `notion:notion-search`     | Resolve a named parent page to its ID         |
| `notion`   | `notion:notion-create-pages` | Create the new page with structured content |

## When to use this skill

Use this skill when:
- The user wants to create a new Notion page from a description or notes
- The user asks to "write up", "draft", or "create a doc" in Notion
- The user wants to turn meeting notes, a brief, or bullet points into a page

Do NOT use this skill when:
- The user wants to update or append to an existing page (use `/notion-update`)
- The user wants to find an existing page (use `/notion-search`)

## Instructions

### Step 1: Gather inputs
Collect from the user:
- **Title** — the page title (required; infer from context if obvious)
- **Parent location** — a page name, page URL, or page ID to nest the new page
  under. If not provided, ask: *"Where in Notion should this page live?"*
- **Content** — the description, notes, or outline to turn into the page body

### Step 2: Resolve the parent page (if named, not an ID/URL)
If the user provided a parent page name rather than an ID, call
`notion:notion-search` with that name and pick the best match. Confirm with
the user if the match is ambiguous.

### Step 3: Structure the content
Before creating the page, organise the content into logical sections:
- Use headings (H2) for major sections
- Use bulleted lists for items and sub-points
- Use a short paragraph at the top as a summary or context note if appropriate

### Step 4: Create the page
Call `notion:notion-create-pages` with:
- `title`: the page title
- `parent`: the resolved parent page ID
- The structured content blocks from Step 3

### Step 5: Confirm and return the link
Return the new page title and direct Notion link. Offer to search for related
pages or further update the new page.

## Keywords
create notion page, draft notion doc, write notion page, new notion doc, make a notion page, add to notion
