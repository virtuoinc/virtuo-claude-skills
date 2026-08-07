---
name: gtm-digest
description: >
  Generates a weekly GTM executive summary by pulling data from HubSpot (pipeline health,
  recent closed deals, campaign performance) and Granola (customer meeting themes, commitments,
  and competitor mentions). Synthesizes wins, risks, funnel highlights, campaign results, and
  meeting patterns into a single brief. Use when asked for a GTM digest, weekly GTM summary,
  or revenue team update. Invoke with /gtm-digest.
---

# GTM Digest

## MCP Requirements

| MCP Server | Tool                                    | Purpose                                              |
|------------|-----------------------------------------|------------------------------------------------------|
| `hubspot`  | `hubspot-search_crm_objects`            | Fetch New Broker Sales pipeline snapshot and recent won/lost deals |
| `hubspot`  | `hubspot-read_campaign_data`            | Fetch campaign-level performance metrics             |
| `hubspot`  | `hubspot-get_content_analytics_report`  | Fetch email engagement stats for the period          |
| `granola`  | `granola-list_meetings`                 | List customer meetings recorded in the past week     |
| `granola`  | `granola-query_granola_meetings`        | Identify themes, commitments, and competitor mentions|

## When to use this skill

Use this skill when:
- The user asks for a "GTM digest", "weekly GTM summary", or "revenue team update"
- The user asks "how did we do this week" or "what's the GTM status"
- The user wants a cross-channel view combining pipeline, marketing, and meeting activity
- A team lead wants a single brief before a weekly GTM review meeting

Do NOT use this skill when:
- The user wants a deep dive into one area — use `/pipeline-review`, `/funnel-metrics`, or `/campaign-performance` instead

## Instructions

### Step 1: Determine the reporting window

Default to the last 7 days. Confirm with the user or adjust to a different period if specified (e.g. last month, this quarter).

### Step 2: Fetch all data sources in parallel

Run all five calls simultaneously:

**Call A** — `hubspot-search_crm_objects` for the **open pipeline snapshot**:
- `objectType`: `deals`
- `filterGroups`: `pipeline` = `"811852437"` (New Broker Sales); exclude `closedwon` and `closedlost` stages
- `properties`: `["dealname", "dealstage", "amount", "closedate", "hubspot_owner_id", "hs_lastmodifieddate"]`
- `limit`: 200

**Call B** — `hubspot-search_crm_objects` for **deals closed in the reporting window**:
- `objectType`: `deals`
- `filterGroups`: `closedate` within the window; `dealstage` is `closedwon` OR `closedlost`; `pipeline` = `"811852437"` (New Broker Sales)
- `properties`: `["dealname", "dealstage", "amount", "closedate", "hubspot_owner_id"]`
- `limit`: 50

**Call C** — `hubspot-read_campaign_data`:
- Fetch `metrics` view for all campaigns active during the reporting window

**Call D** — `hubspot-get_content_analytics_report`:
- Fetch aggregate stats for emails sent within the reporting window

**Call E** — `granola-list_meetings`:
- Time range: start and end of the reporting window
- Returns meeting titles and metadata for all recorded meetings

### Step 3: Surface meeting themes from Granola

Using the meeting list from Call E, call `granola-query_granola_meetings` with the following natural-language queries (run in parallel):
- "What objections or concerns came up in customer or sales calls this week?"
- "What follow-ups or next steps were committed to in meetings this week?"
- "Were any competitors mentioned in meetings this week?"

Extract top themes, commitments, and competitor names from the cited responses.

### Step 4: Flag at-risk deals

From Call A results, flag deals as at-risk if:
- `hs_lastmodifieddate` is more than 14 days ago (stalled), OR
- `closedate` is in the past (overdue)

### Step 5: Synthesize and present the digest

---

## GTM Digest — Week of [start date]

### 🏆 Wins
- Deals closed won: [list deal names, amounts, owners]
- Total won revenue this period: **$X**
- Top campaign highlight: [name — key metric]

### 📊 Pipeline Health
- Open pipeline: **$X across N deals**
- Stage breakdown: [Stage: N deals ($X) — one line per stage]
- At-risk deals: **N deals ($X total value)** — stalled or overdue

### 📧 Campaign & Email Highlights
- [Campaign name]: [top metric — e.g. "1,200 contacts reached, $X attributed revenue"]
- Best email: [name] — [open rate]% open rate, [CTOR]% CTOR
- Health alerts: [any emails flagged for high bounces or unsubscribes, or "None"]

### 🤝 Meeting Themes (from Granola)
- [N] customer meetings recorded this week
- **Top themes**: [bullet list — max 3]
- **Key commitments made**: [bullet list of follow-ups/next steps from meeting notes]
- **Competitor mentions**: [list, or "None noted"]

### ⚠️ Risks & Watch Items
- [Stalled deals and owner names]
- [Campaign health alerts]
- [Meeting patterns suggesting deal risk or churn signals]

---

Offer to drill into any section with `/pipeline-review`, `/funnel-metrics`, or `/campaign-performance`.

## Keywords

GTM digest, weekly GTM summary, revenue update, GTM review, weekly pipeline summary, team digest, GTM status, revenue team update, weekly summary, go to market digest
