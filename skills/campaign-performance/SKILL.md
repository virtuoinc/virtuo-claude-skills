---
name: campaign-performance
description: >
  Pulls HubSpot campaign analytics and marketing email performance — opens, clicks, bounces,
  revenue attribution, and asset-level metrics. Scores each campaign and surfaces
  recommendations. Use when asked for campaign performance, email stats, marketing ROI,
  attribution data, or campaign health. Invoke with /campaign-performance.
---

# Campaign Performance

## MCP Requirements

| MCP Server | Tool                                  | Purpose                                              |
|------------|---------------------------------------|------------------------------------------------------|
| `Hubspot`  | `Hubspot:get_campaign_analytics`      | Fetch campaign-level metrics and revenue attribution |
| `Hubspot`  | `Hubspot:get_campaign_asset_metrics`  | Fetch per-asset metrics for emails, pages, and CTAs  |
| `Hubspot`  | `Hubspot:get_marketing_email_analytics` | Fetch email open/click/bounce rates and health     |
| `Hubspot`  | `Hubspot:search_crm_objects`          | Fetch deals attributed to a campaign (optional)      |

## When to use this skill

Use this skill when:
- The user asks how a campaign is performing
- The user asks for email open rates, click rates, or bounce rates
- The user asks for marketing attribution or campaign ROI
- The user asks "what marketing is working" or "which campaigns are driving revenue"
- The user asks for a campaign health check or email performance report

Do NOT use this skill when:
- The user wants deal pipeline or funnel data (use `/pipeline-review` or `/funnel-metrics`)
- The user wants a combined cross-channel summary (use `/gtm-digest`)

## Instructions

### Step 1: Identify the campaign(s)

If the user named a specific campaign, use that as the target. If they want an overview, fetch metrics across all active campaigns.

Ask if a time filter is needed (e.g. campaigns active in the last 30 days).

### Step 2: Fetch campaign analytics (run all calls in parallel)

**Call A** — `Hubspot:get_campaign_analytics` with:
- `campaignId`: the campaign ID (or list of IDs for overview)
- `reportingView`: `metrics`

This returns contacts reached, sessions, impressions, and engagement counts.

**Call B** — `Hubspot:get_campaign_analytics` with the same campaign(s):
- `reportingView`: `revenue-attribution`

This returns attributed new contacts, influenced contacts, attributed deals, and revenue.

**Call C** — `Hubspot:get_campaign_asset_metrics` with:
- `campaignId`: the campaign ID
- Retrieve metrics for all available asset types (emails, landing pages, CTAs)

**Call D** — `Hubspot:get_marketing_email_analytics`:
- Filter for emails linked to this campaign (use email asset IDs from Call C)
- Request aggregate stats: `delivered`, `opened`, `clicked`, `bounced`, `unsubscribed`

### Step 3: Score assets and flag health issues

From Calls C and D:
- Rank email assets by open rate and click-to-open rate (CTOR = clicks ÷ opens)
- Flag emails as **⚠ Unhealthy** if: bounce rate >2% OR unsubscribe rate >0.5%
- Identify top-performing landing pages by conversion rate
- Note any asset with zero engagement

### Step 4: Present the campaign scorecard

---

## Campaign Performance: [Campaign Name]

### Overview

| Metric | Value |
|--------|-------|
| Contacts Reached | N |
| Sessions Generated | N |
| New Contacts Attributed | N |
| Attributed Deals | N |
| Attributed Revenue | $X |

### Email Health

| Email | Delivered | Open Rate | CTOR | Bounce Rate | Status |
|-------|-----------|-----------|------|-------------|--------|
| [Name] | N | X% | X% | X% | ✓ Healthy / ⚠ Unhealthy |

### Top Assets

List top 3 assets by engagement with key stat (open rate, click rate, or conversion rate).

### Underperformers

Flag assets with zero clicks, high unsubscribes, or bounce rate above threshold.

### Recommendations

Surface 2–3 specific, actionable recommendations based on the data, for example:
- Re-send to unopened segment if open rate <20%
- A/B test subject line on low-CTOR emails
- Pause or investigate high-bounce emails before next send
- Promote top-performing landing page in follow-up sequences

---

Offer to drill into a specific email or asset, or suggest `/gtm-digest` for a cross-channel summary.

## Keywords

campaign performance, email analytics, open rate, click rate, bounce rate, marketing ROI, campaign attribution, campaign health, email stats, marketing performance, email performance, campaign metrics
