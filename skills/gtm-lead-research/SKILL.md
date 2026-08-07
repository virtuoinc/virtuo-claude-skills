---
name: gtm-lead-research
description: >
  Researches a prospect or lead by pulling their HubSpot contact and deal history,
  checking Granola for any prior meeting notes, and running web searches for company
  overview, recent news, funding, and key people. Produces a structured lead profile
  with ICP fit signals and a suggested outreach angle. Use when asked to research a lead,
  prospect, or company before first contact or during qualification. Invoke with /gtm-lead-research.
---

# Lead Research

## MCP Requirements

| MCP Server   | Tool                             | Purpose                                                           |
|--------------|----------------------------------|-------------------------------------------------------------------|
| `hubspot`    | `hubspot-search_crm_objects`     | Find existing contact record and deals in New Broker Sales pipeline |
| `hubspot`    | `hubspot-query_crm_data`         | Query prior engagement records (emails, calls, notes) for the contact |
| `granola`    | `granola-query_granola_meetings` | Check for any recorded past meetings with this person             |
| `Web search` | `web_search`                     | Research company overview, news, funding, key people              |

## When to use this skill

Use this skill when:
- The user asks to "research a lead", "look up a prospect", or "tell me about [company/person]"
- The user is preparing for first outreach and wants background on a prospect
- The user wants to qualify a lead against ICP before investing time
- The user asks "have we talked to [company] before?" and wants full context

Do NOT use this skill when:
- The user has a meeting already scheduled (use `/deal-prep` instead)
- The user wants pipeline or funnel metrics (use `/pipeline-review` or `/funnel-metrics`)

## Instructions

### Step 1: Identify the prospect

Extract from the user's request:
- **Contact name** and/or **email address**
- **Company name**

If only a company name is given, proceed without a contact name. If only a name is given without a company, ask.

### Step 2: Look up HubSpot and Granola history (run all in parallel)

**Call A** — `hubspot-search_crm_objects` for the contact:
- `objectType`: `contacts`
- Filter by email if available; otherwise by `firstname` + `lastname` or company name
- `properties`: `["firstname", "lastname", "email", "company", "jobtitle", "hs_lead_status", "hubspot_owner_id", "createdate", "notes_last_contacted"]`

**Call B** — `hubspot-search_crm_objects` for associated deals (run after Call A if a contact is found, otherwise skip):
- `objectType`: `deals`
- Filter by associated contact ID; `pipeline` = `"811852437"` (New Broker Sales)
- `properties`: `["dealname", "dealstage", "amount", "closedate", "hs_lastmodifieddate"]`

**Call C** — `hubspot-query_crm_data`:
- Query engagement records (emails, calls, notes) for the contact by email or company name

**Call D** — `granola-query_granola_meetings`:
- Query: "Have we met with [contact name] or [company name]?"

### Step 3: Run web research (run all queries in parallel)

Call `web_search` for each of the following:
- `"[Company name]" overview product customers`
- `"[Company name]" funding news site:techcrunch.com OR site:crunchbase.com`
- `"[Company name]" recent news [current year]`
- `"[Contact name]" "[Company name]" role title`

Extract: what the company does, funding stage, recent news or leadership changes, and the contact's seniority.

### Step 4: Assess ICP fit

Based on gathered data, assess:
- **Prior engagement**: existing contact? past deal? how far did it get?
- **Meeting history**: have we spoken before? what was the outcome?
- **Company signals**: funding stage, growth signals, hiring in relevant areas
- **Timing**: recent trigger events (funding round, product launch, leadership change)

### Step 5: Present the lead profile

---

## Lead Profile: [Contact Name] — [Company]

### Company Overview
[2–3 sentence summary: what they do, who their customers are, company stage]

### Key Person
**[Name]** — [Title]
[1–2 sentences on their role and likely influence in a buying decision]

### HubSpot History
- **Contact status**: [New / Existing — created date, owner]
- **Past deals**: [deal names, stages, and outcomes — or "No prior deals found"]
- **Prior comms**: [summary of logged conversations — or "None on record"]

### Prior Meetings (Granola)
[Summary of any recorded meetings and key themes — or "No meetings on record"]

### Company Signals
- **Funding**: [stage and latest round if known]
- **Recent news**: [1–2 relevant items]
- **Hiring signals**: [relevant job postings if found]

### ICP Fit
[Strong fit / Potential fit / Low fit — with 1–2 key reasons]

### Suggested Outreach Angle
[1–3 specific, personalised talking points based on the data — e.g. tie to recent funding, reference a pain point from job postings, or follow up on a prior conversation thread]

---

Offer to run `/deal-prep` if a meeting with this prospect is already scheduled.

## Keywords

lead research, prospect research, company research, look up lead, qualify prospect, ICP fit, outreach prep, who is, tell me about, background on, research company, prospect background
