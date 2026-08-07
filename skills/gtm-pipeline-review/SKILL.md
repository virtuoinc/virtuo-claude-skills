---
name: gtm-pipeline-review
description: >
  Pulls a live snapshot of the HubSpot deal pipeline — shows deals grouped by stage,
  flags at-risk deals with no recent activity or past their close date, and breaks down
  the pipeline by owner. Use when asked to review the pipeline, check deal stages, find
  stuck deals, or get a pipeline health summary. Invoke with /gtm-pipeline-review.
---

# GTM Pipeline Review

## MCP Requirements

| MCP Server | Tool                           | Purpose                                                   |
|------------|--------------------------------|-----------------------------------------------------------|
| `hubspot`  | `hubspot-search_crm_objects`   | Fetch open deals in the New Broker Sales pipeline         |
| `hubspot`  | `hubspot-search_owners`        | Resolve owner names from IDs                              |
| `hubspot`  | `hubspot-get_properties`       | Resolve deal stage enum labels (optional)                 |

## When to use this skill

Use this skill when:
- The user asks to "review the pipeline", "show me the pipeline", or "pipeline health check"
- The user asks what deals are in a particular stage
- The user asks about stuck, stalled, or at-risk deals
- The user asks for a pipeline breakdown by owner or stage

Do NOT use this skill when:
- The user wants funnel conversion rates or win rates (use `/funnel-metrics`)
- The user wants campaign or email analytics (use `/campaign-performance`)

## Instructions

### Step 1: Determine scope

This skill always scopes to the **New Broker Sales** pipeline (ID: `811852437`). Infer from context, or ask the user, whether they want:
- **All open deals** (default)
- A specific owner's deals
- Deals closing within a specific window (e.g. this quarter)

### Step 2: Fetch open deals

Call `hubspot-search_crm_objects` with:
- `objectType`: `deals`
- `filterGroups`: `pipeline` = `"811852437"` (New Broker Sales); exclude `closedwon` and `closedlost` stages
- `properties`: `["dealname", "dealstage", "amount", "closedate", "hubspot_owner_id", "hs_lastmodifieddate", "hs_activity_count"]`
- `limit`: 200

If the user specified an owner, add a filter for `hubspot_owner_id`.
If the user specified a close date window, add a filter for `closedate` within that range.

### Step 3: Resolve owner names and stage labels (run in parallel)

- Call `hubspot-search_owners` to map all owner IDs present in the results to display names.
- If stage labels are not self-explanatory, call `hubspot-get_properties` for `objectType: deals`, `propertyName: dealstage` to get the ordered stage list and labels.

### Step 4: Analyse and flag deals

Group deals by `dealstage`. For each deal, flag it as **at-risk** if either:
- `hs_lastmodifieddate` is more than 14 days ago (stalled)
- `closedate` is in the past (overdue)

Sum `amount` per stage for total pipeline value.

### Step 5: Present the pipeline snapshot

---

## Pipeline Snapshot — [today's date]

### By Stage

| Stage | # Deals | Pipeline Value | At-Risk |
|-------|---------|----------------|---------|
| [Stage name] | N | $X | N |

**Total open pipeline: $X across N deals**

### At-Risk Deals

For each flagged deal:
- **[Deal name]** — [Owner] — Stage: [stage]
  - Close date: [date] | Last activity: [X days ago]
  - Recommended action: follow up / re-evaluate close date / escalate

### By Owner

| Owner | Open Deals | Pipeline Value |
|-------|-----------|----------------|
| [Name] | N | $X |

---

Offer to drill into a specific stage or owner, or suggest running `/funnel-metrics` for conversion analysis.

## Keywords

pipeline review, pipeline health, deal stages, stuck deals, at-risk deals, pipeline snapshot, stalled deals, pipeline by owner, open deals, deal pipeline
