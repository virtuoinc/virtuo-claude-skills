---
name: gtm-deal-prep
description: >
  Prepares a sales-focused briefing before a customer or prospect meeting — pulls the HubSpot
  deal and contact context, retrieves past meeting themes and open commitments from Granola,
  and runs web research on the company if it is a first meeting or the deal context is stale.
  Produces a briefing card with deal status, relationship history, and talking points mapped to
  the current deal stage. Use when preparing for a sales call, discovery call, demo, or any
  external meeting with a prospect or customer. Invoke with /gtm-deal-prep.
---

# Deal Prep

## MCP Requirements

| MCP Server      | Tool                                    | Purpose                                                              |
|-----------------|-----------------------------------------|----------------------------------------------------------------------|
| `Microsoft 365` | `Calendars.Read`                        | Find the upcoming meeting if not specified by the user               |
| `hubspot`       | `hubspot-search_crm_objects`            | Look up the contact record and open deal in the New Broker Sales pipeline |
| `hubspot`       | `hubspot-query_crm_data`                | Query engagement records (emails, calls, notes) for the contact      |
| `granola`       | `granola-query_granola_meetings`        | Pull themes and commitments from past recorded meetings              |
| `Web search`    | `web_search`                            | Research the company if first meeting or stale context               |

## When to use this skill

Use this skill when:
- The user asks to "prep for" or "brief me before" a sales or customer call
- The user asks "what do I need to know before my call with [company/person]?"
- The user asks for deal context before a demo, discovery, or follow-up meeting
- The user asks "what did we commit to last time we spoke with [company]?"

Do NOT use this skill when:
- The meeting is internal with colleagues (use `/meeting-prep` instead)
- No meeting is scheduled yet and the user wants to research a prospect (use `/lead-research`)
- The user wants pipeline-wide metrics (use `/pipeline-review` or `/funnel-metrics`)

## Instructions

### Step 1: Identify the meeting and prospect

If the user named a specific company or person, use that directly.

Otherwise, call `Calendars.Read` to find the next upcoming external meeting. Extract the external attendee email(s) and company name from the event. Ask the user to confirm if multiple external meetings are returned.

### Step 2: Look up CRM and conversation history (run all in parallel)

**Call A** — `hubspot-search_crm_objects` for the contact:
- `objectType`: `contacts`
- Filter by attendee email address
- `properties`: `["firstname", "lastname", "email", "company", "jobtitle", "hs_lead_status", "hubspot_owner_id", "notes_last_contacted"]`

**Call B** — `hubspot-search_crm_objects` for the associated deal:
- `objectType`: `deals`
- Filter by associated contact ID (from Call A); if no contact found, filter by company name
- Filter `pipeline` = `"811852437"` (New Broker Sales — the only pipeline in scope)
- `properties`: `["dealname", "dealstage", "amount", "closedate", "hs_lastmodifieddate", "hubspot_owner_id", "description"]`
- Sort by `hs_lastmodifieddate` descending, limit 3

**Call C** — `hubspot-query_crm_data`:
- Query engagement records (emails, calls, notes) for the contact by email address

**Call D** — `granola-query_granola_meetings`:
- Query: "What happened in our meetings with [contact name] or [company name]?"

### Step 3: Run web research if needed

Run `web_search` if **any** of the following are true:
- No Granola meetings were found (first meeting)
- The most recent HubSpot deal activity is more than 60 days ago
- The user explicitly asked for company background

Run these queries in parallel:
- `"[Company name]" overview product customers`
- `"[Company name]" recent news funding [current year]`
- `"[Contact name]" "[Company name]" role seniority`

### Step 4: Identify open commitments

From the Granola results, extract:
- Things *we* promised to send, investigate, or follow up on
- Questions the prospect raised that were left unanswered
- Next steps that were agreed in previous meetings

Cross-reference with HubSpot logged notes to check if any were already actioned.

### Step 5: Map talking points to deal stage

Tailor the suggested talking points to the current HubSpot deal stage:
- **New / Discovery**: ICP validation questions, pain discovery, qualification
- **Demo / Evaluation**: address known objections, show relevant use cases
- **Proposal / Negotiation**: ROI framing, urgency, resolve blockers
- **Closing**: confirm outstanding concerns and next steps to sign

### Step 6: Present the sales briefing card

---

## Deal Prep: [Company] — [Meeting date & time]

### Deal Status
- **Deal**: [Deal name] — Stage: [stage] — Value: $X — Close date: [date]
- **Owner**: [name] | **Contact**: [name, title]
- **Last CRM activity**: [X days ago — brief description]

### Relationship History
[2–3 sentence summary: how long in the pipeline, number of touchpoints, where things stand]

### Past Meeting Themes (Granola)
[Bullet list of key themes, objections raised, and decisions from prior meetings — or "First meeting — no prior notes found"]

### Open Commitments
[Bullet list of outstanding follow-ups from prior meetings — or "None identified"]

### Company Context *(if web research was run)*
[Company overview, recent news, funding signals]

### Suggested Talking Points
Based on deal stage ([stage]) and relationship history:
1. [Specific, personalised talking point]
2. [Specific, personalised talking point]
3. [Specific, personalised talking point]

---

Offer to run `/lead-research` for a deeper company profile, or `/pipeline-review` to see where this deal sits in the overall pipeline.

## Keywords

deal prep, sales prep, call prep, demo prep, discovery prep, prep for call, brief before call, customer meeting prep, prospect meeting, what to know before, sales call, customer call, external meeting prep
