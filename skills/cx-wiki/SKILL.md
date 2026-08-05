---
name: cx-wiki
description: >
  Answers Customer Experience questions by searching the CX Knowledge Wiki in Notion.
  Covers geographical process nuances, available providers, and service availability by region.
  Answers are strictly scoped to the CX wiki — if information is not found there, the skill
  explicitly says so rather than guessing or searching externally. Invoke with /cx-wiki.
---

# CX Knowledge Wiki

## MCP Requirements

| MCP Server | Tool                   | Purpose                                              |
|------------|------------------------|------------------------------------------------------|
| `notion`   | `notion:notion-search` | Search the CX wiki for pages matching the query      |
| `notion`   | `notion:notion-fetch`  | Retrieve the content of matching wiki pages          |

## When to use this skill

Use this skill when:
- A CX team member asks how a process works in a specific country or region
- A CX team member asks which providers or services are available in a geography
- A CX team member asks about policy, procedure, or operational nuances for a region
- The user asks "what does the CX wiki say about X" or "how do we handle X in [country]"

Do NOT use this skill when:
- The user wants to find a new or external provider not in the Virtuo network (use `/cx-provider-search`)
- The question is about a non-CX topic

## Instructions

The CX Knowledge Wiki root page ID is `1a8c2928f48a80abb3a0e5d59e203d08`.

### Step 1: Extract the query topic

Identify from the user's request:
- **Topic**: the process, policy, or service type (e.g. "roadside assistance", "refund process", "rental extensions")
- **Geography**: the country or region if mentioned (e.g. "France", "DACH", "UK")

If both are unclear, ask one short clarifying question before searching.

### Step 2: Search the CX wiki

Call `notion:notion-search` with a query combining the topic and geography:
- Use the format: `"[topic] [geography]"` — e.g. `"roadside assistance France"`
- If no geography was given, search by topic only

From the results, identify pages that belong to the CX Knowledge Wiki by checking whether their breadcrumb or parent reference matches the wiki root page ID (`1a8c2928f48a80abb3a0e5d59e203d08`). Discard results from unrelated Notion spaces.

If no relevant results are returned, proceed to Step 4.

### Step 3: Retrieve page content

For each relevant result (up to 3), call `notion:notion-fetch` with the page ID to retrieve its full content.

If the user's question spans multiple sub-topics, fetch each relevant page in parallel.

### Step 4: Answer the question

Using only the retrieved wiki content:
- Answer the user's question directly and concisely
- Cite the source page title and link for every claim made
- If the content partially answers the question, state clearly what is covered and what is not

If no relevant content was found in the CX wiki, respond:
> "I couldn't find information about this in the CX Knowledge Wiki. For provider lookups, try `/cx-provider-search`. For anything else, check the wiki directly or ask your team lead."

Do NOT invent answers, speculate, or search the web. The CX wiki is the only source of truth for this skill.

## Output format

---

**[Topic] — [Geography if applicable]**

[Direct answer based on wiki content]

**Source**: [Page title](page URL) *(and additional sources if multiple pages were used)*

---

## Keywords

cx wiki, cx knowledge base, how do we handle, process in, providers in, services in, available in, policy for, geographic nuances, country process, region process, how does CX handle
