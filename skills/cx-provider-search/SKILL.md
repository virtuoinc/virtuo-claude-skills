---
name: cx-provider-search
description: >
  Finds service providers for a given geography and service type by first checking
  Virtuo-approved providers in the CX Knowledge Wiki, then supplementing with web search
  for any gaps. Returns known Virtuo providers separately from external options so CX teams
  can prioritise approved partners. Invoke with /cx-provider-search.
---

# CX Provider Search

## MCP Requirements

| MCP Server   | Tool                     | Purpose                                                                      |
|--------------|--------------------------|------------------------------------------------------------------------------|
| `notion`     | `notion:notion-search`   | Search the CX wiki for known Virtuo providers in the region                  |
| `notion`     | `notion:notion-fetch`    | Retrieve provider details from CX wiki pages                                 |
| `Web search` | `Web search:web_search`  | Find actual providers, their contact details, and review scores              |

## When to use this skill

Use this skill when:
- A CX team member needs to find a provider (e.g. roadside assistance, repair shop, rental partner) in a specific country or city
- A CX team member asks "who do we work with in [geography]" or "find a [service] provider in [location]"
- The user needs to identify provider options for an active customer case

Do NOT use this skill when:
- The user has a process or policy question about a region (use `/cx-wiki`)
- The user is not looking for an external service provider

## Instructions

The CX Knowledge Wiki root page ID is `1a8c2928f48a80abb3a0e5d59e203d08`.

### Step 1: Extract search parameters

Identify from the user's request:
- **Service type**: what kind of provider is needed (e.g. "roadside assistance", "bodywork repair", "towing", "replacement vehicle")
- **Geography**: country, region, or city

If either is missing, ask one short clarifying question before proceeding.

### Step 2: Check the CX wiki for known providers (run in parallel)

**Call A** — `notion:notion-search` with query `"[service type] [geography] providers"`:
- From results, keep only pages that belong to the CX Knowledge Wiki (parent matches root ID `1a8c2928f48a80abb3a0e5d59e203d08`)

**Call B** — `notion:notion-fetch` on the CX wiki root page (`1a8c2928f48a80abb3a0e5d59e203d08`):
- Scan child pages for a providers or services directory section
- If a relevant child page is found, note its ID for fetching in Step 3

### Step 3: Retrieve provider details from wiki

For any relevant pages identified in Step 2, call `notion:notion-fetch` to retrieve their full content (run in parallel if multiple).

Extract: provider names, contact details, coverage area, and any notes on approval status or preferred-partner designation.

### Step 4: Web search for gaps

If the wiki returned no providers for the requested geography and service type, or if the user explicitly wants additional options, run the following web searches (in parallel where possible):

**Search A — Find actual providers:**
- Query: `"[service type] company [geography]"` — e.g. `"roadside assistance company Spain"`
- Exclude aggregators, directories, and marketplaces (e.g. Yelp lists, Angi, Bark, Thumbtack, HomeAdvisor). Only surface individual businesses that directly provide the service.

**Search B — Contact details:**
For each candidate provider found, search `"[provider name] phone email contact [geography]"` to retrieve their website, phone number, and email address.

**Search C — Review scores:**
For each provider, search `"[provider name] [geography] reviews"` to find:
- Google Places star rating and review count
- Scores from review sites such as Trustpilot, Yelp, Homestars, or similar local platforms
- A brief summary of recurring themes in reviews (e.g. "Praised for fast response times; some complaints about billing")

Only include providers where at least a website or phone number can be confirmed. Discard any result that is an aggregator, lead-generation site, or comparison platform rather than an actual service provider.

### Step 5: Present results

Separate the two categories clearly:

---

## Provider Search: [Service Type] in [Geography]

### Virtuo-Approved Providers
*(from the CX Knowledge Wiki)*

| Provider | Website | Phone | Email | Coverage | Notes | Source |
|----------|---------|-------|-------|----------|-------|--------|
| [Name] | [URL] | [Phone] | [Email] | [Area] | [Preferred partner / approved / etc.] | [Wiki page link] |

*If none found: "No Virtuo-approved providers found in the CX wiki for this geography and service type."*

### Additional Options
*(from web search — not yet Virtuo-approved)*

For each provider, include a contact block and review summary:

**[Provider Name]**
- **Website:** [URL]
- **Phone:** [number or "not found"]
- **Email:** [address or "not found"]
- **Coverage:** [area]
- **Reviews:** [Google ★X.X (N reviews)] | [Yelp / Trustpilot / Homestars: X/5 if available]
- **Summary:** [2–3 sentence synthesis of what customers say — strengths and any notable concerns]

---

If only wiki results exist and they fully answer the question, skip the web search section. Do not present external options as equivalent to approved providers. Never include aggregators, directories, or lead-generation platforms in the results.

## Keywords

find provider, provider in, who do we work with, roadside assistance, repair shop, towing provider, rental partner, service provider, find a partner in, coverage in, provider search
