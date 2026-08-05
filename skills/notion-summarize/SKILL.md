---
name: notion-summarize
description: >
  Fetches a Notion page by URL or title and returns a structured summary
  covering key points, decisions, owners, and next steps. When a title is
  provided instead of a URL, searches the workspace first to resolve the page.
  Invoke with /notion-summarize.
---

# Notion Summarize

## MCP Requirements

| MCP Server | Tool                       | Purpose                                          |
|------------|----------------------------|--------------------------------------------------|
| `notion`   | `notion:notion-search`     | Find a page by title when no URL is given        |
| `notion`   | `notion:notion-fetch`      | Retrieve full page content for summarization     |

## When to use this skill

Use this skill when:
- The user shares a Notion URL and asks for a summary or TL;DR
- The user asks "what does the [page name] doc say"
- The user wants to quickly understand a page without reading it fully
- The user asks for key decisions, owners, or next steps from a doc

Do NOT use this skill when:
- The user wants to find which pages exist on a topic (use `/notion-search`)
- The user wants to edit or update the page (use `/notion-update`)

## Instructions

### Step 1: Resolve the page
- **URL provided**: proceed directly to Step 2.
- **Title provided**: call `notion:notion-search` with the title. Pick the
  top match. If multiple close matches exist, list them and ask the user to
  confirm.

### Step 2: Fetch the page content
Call `notion:notion-fetch` with the resolved page URL or ID to retrieve the
full page content.

### Step 3: Summarize
Produce a structured summary:
- **Overview** — 2–3 sentence description of what the page is about
- **Key points** — bullet list of the most important facts, decisions, or content
- **Owners / DRIs** — names or roles mentioned as responsible (if present)
- **Next steps / action items** — any tasks or follow-ups called out (if present)
- **Last updated** — date and author if available

Omit sections that have no relevant content in the page.

### Step 4: Offer follow-up actions
After the summary, offer to:
- Open or link to the full page
- Update the page with new information (`/notion-update`)
- Search for related pages (`/notion-search`)

## Keywords
summarize notion page, tldr notion, what does notion say, read notion doc, notion page summary, explain notion doc
