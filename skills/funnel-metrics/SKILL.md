---
name: funnel-metrics
description: >
  Calculates GTM funnel metrics from HubSpot — stage-by-stage conversion rates, win rate,
  average deal velocity, and a revenue forecast weighted by historical conversion rates.
  Use when asked for funnel performance, conversion rates, win/loss analysis, deal velocity,
  or revenue forecasting. Invoke with /funnel-metrics.
---

# Funnel Metrics

## MCP Requirements

| MCP Server | Tool                         | Purpose                                              |
|------------|------------------------------|------------------------------------------------------|
| `Hubspot`  | `Hubspot:search_crm_objects` | Fetch deals within a time period for analysis        |
| `Hubspot`  | `Hubspot:get_properties`     | Resolve deal stage enum values and their order       |

## When to use this skill

Use this skill when:
- The user asks for conversion rates, win rate, or funnel performance
- The user asks how deals are converting through the pipeline
- The user asks for deal velocity or average deal cycle length
- The user asks for a revenue forecast or projected close revenue
- The user asks "how is the team performing" or "what's our win rate"

Do NOT use this skill when:
- The user wants a live view of current open deals (use `/pipeline-review`)
- The user wants campaign or email analytics (use `/campaign-performance`)

## Instructions

### Step 1: Determine the analysis window

Default to the last 90 days if no period is specified. Ask if the user wants to filter by a specific owner or team.

### Step 2: Fetch stage definitions and deals (run in parallel)

**Call A** — `Hubspot:get_properties` with:
- `objectType`: `deals`
- `propertyName`: `dealstage`

Extract the ordered list of stage labels and internal values. This defines the funnel structure and the order of stages.

**Call B** — `Hubspot:search_crm_objects` for closed deals:
- `objectType`: `deals`
- `filterGroups`: `closedate` within the analysis window; `dealstage` is `closedwon` OR `closedlost`
- `properties`: `["dealname", "dealstage", "amount", "closedate", "createdate", "hubspot_owner_id"]`
- `limit`: 200

**Call C** — `Hubspot:search_crm_objects` for all deals created in the period:
- `objectType`: `deals`
- `filterGroups`: `createdate` within the analysis window
- `properties`: `["dealname", "dealstage", "amount", "createdate", "hubspot_owner_id"]`
- `limit`: 200

### Step 3: Calculate metrics

**Win rate**: `closedwon count ÷ (closedwon + closedlost count) × 100`

**Stage conversion rates**: For each adjacent stage pair (Stage N → Stage N+1), count deals from Call C that reached at least Stage N+1 vs. those that entered Stage N.

**Average deal velocity**: For won deals only, compute `mean(closedate − createdate)` in days.

**Revenue forecast**: Sum `amount` for all currently open deals in Call C, weighted by each deal's stage historical conversion rate.

### Step 4: Present funnel metrics

---

## Funnel Metrics — [period]

### Conversion Funnel

| Stage | Deals Entered | Converted to Next | Conversion Rate |
|-------|--------------|-------------------|----------------|
| [Stage] | N | N | X% |

**Overall win rate: X%**

### Key Metrics

| Metric | Value |
|--------|-------|
| Win Rate | X% |
| Avg Deal Velocity (won) | X days |
| Total Won Revenue | $X |
| Total Lost Revenue | $X |
| Deals Analysed | N |

### Revenue Forecast

Estimated close revenue from current open pipeline: **$X**
*(Stage-weighted by historical conversion rates from this period)*

---

Offer to break down metrics by owner, or suggest `/pipeline-review` for the live pipeline view.

## Keywords

funnel metrics, conversion rates, win rate, deal velocity, funnel analysis, stage conversion, revenue forecast, pipeline performance, win loss, sales performance, funnel performance
