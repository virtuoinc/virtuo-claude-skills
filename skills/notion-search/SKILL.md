---
name: notion-search
description: >
  Searches the Notion workspace for pages, docs, and wiki content matching a
  keyword, topic, or title. Returns ranked results with title, parent space,
  last edited date, and direct link. When no query is given, surfaces recently
  viewed pages instead. Invoke with /notion-search.
---

# Notion Search

## MCP Requirements

| MCP Server | Tool                              | Purpose                                        |
|------------|-----------------------------------|------------------------------------------------|
| `notion`   | `notion:notion-search`            | Search workspace by keyword or topic           |
| `notion`   | `notion:notion-list-recent-pages` | Surface recent pages when no query is given    |
| `notion`   | `notion:notion-fetch`             | Read a result page for a short excerpt (optional) |

## When to use this skill

Use this skill when:
- The user asks to find, search for, or look up a Notion page or document
- The user wants to know what Notion docs exist on a topic or project
- The user asks "what did we write about X" or "is there a Notion page for Y"
- The user asks for their recent or favourite Notion pages with no specific query

Do NOT use this skill when:
- The user wants to read or summarize a specific page they already have a link to (use `/notion-summarize`)
- The user wants to create or update a page (use `/notion-draft` or `/notion-update`)

## Instructions

### Step 1: Clarify the query
If the user provides a clear keyword or topic, proceed. If the request is vague
(e.g. "find my Notion stuff"), ask one short clarifying question about what
topic or project they are looking for.

If the user asks for "recent pages" with no query, skip to Step 2b.

### Step 2a: Search by keyword
Call `notion:notion-search` with the extracted keyword or topic.

### Step 2b: Recent pages fallback
If no query was given, call `notion:notion-list-recent-pages` to return the
user's most recently visited pages.

### Step 3: Present results
Return up to 10 results in a ranked list. For each result show:
- **Title** (linked)
- Parent space or page
- Last edited date

If fewer than 3 results are returned, offer to broaden the search with a
different keyword.

### Step 4: Offer to open or summarize
Ask if the user wants to summarize any of the results. If yes, proceed with
the `/notion-summarize` flow by calling `notion:notion-fetch` on the chosen page.

## Keywords
find notion page, search notion, look up notion, notion docs, notion wiki, find wiki page, what does notion say about
