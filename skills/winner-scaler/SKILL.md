---
name: winner-scaler
description: "SKILL __Winner Scaler__ — Analyze multiple Meta Ads campaigns, automatically sort ads into Winners / Undertested / Losers based on CPA and spend thresholds, then take action: scale winners into a new campaign, retest undertested ads in a dedicated campaign, and pause losers. Use this skill whenever the user mentions: 'scala i vincitori', 'winner scaler', 'scale winners', 'trova i winner', 'pausa i loser', 'retesta', 'retest ads', 'analizza le campagne e scala', 'CPA alto pausa', 'split winner loser', 'campagna scale', 'budget scaling', 'creative analysis scale', or any request to review campaign performance and redistribute ads based on results."
---

# Winner Scaler

Analyze N campaigns, classify every ad by performance, and take action automatically.

## What it does

Given a list of campaign IDs and two thresholds:

| Category | Rule | Action |
|---|---|---|
| **Winners** | CPA < CPA threshold | → Duplicate into a new **Scale** campaign at higher budget |
| **Undertested** | Total spend < spend threshold | → Duplicate into a new **Retest** campaign with fair budget |
| **Losers** | CPA ≥ CPA threshold AND spend ≥ spend threshold | → **Pause** in the original campaign |

This replaces the manual weekly workflow of opening Ads Manager, exporting data, sorting in Excel, and recreating campaigns by hand.

## What I need from you

1. **Ad account** — Which ad account to analyze
2. **CPA threshold** — Maximum acceptable CPA to qualify as a winner (e.g. €25)
3. **Spend threshold** — Minimum spend to consider an ad "tested" (e.g. €10)
4. **Date range** — Lookback period for performance data (e.g. "last_7d", "last_14d", "last_30d")
5. **Scale budget** — Daily budget for the new Winners campaign
6. **Retest budget** — Daily budget for the new Retest campaign

**Note:** You do NOT need to provide campaign IDs upfront. Step 0 will show you all active campaigns so you can pick which ones to analyze.

### Optional
- **Campaign name prefix** — Default: "Scale - " and "Retest - " + today's date
- **Keep original ads active** — Whether to keep winner/undertested ads running in the original campaigns too (default: no, pause them)

## Workflow

### Step 0 — Campaign discovery

Before anything, show the user what's running in the account so they can choose.

```
Tool: get_campaigns
Account: {ad_account}
Filter: status ACTIVE or PAUSED (with recent delivery)
```

Then pull campaign-level insights for the requested date range:

```
Tool: get_insights
Level: campaign
Fields: campaign_id, campaign_name, objective, spend, actions (purchase), cost_per_action_type (purchase)
Date preset: {date_range}
```

Present a summary table:

| # | Campaign Name | Objective | Spend | Purchases | CPA | Ads |
|---|---|---|---|---|---|---|

**Ask the user: "Which campaigns do you want me to analyze? Give me the numbers."**

Wait for the user to select. Only proceed with the selected campaigns.

This step exists because not all campaigns are suitable — e.g. the user may have dynamic creative campaigns that shouldn't be split this way, or awareness campaigns without purchase optimization.

### Step 1 — Pull performance data

For each campaign selected in Step 0:

```
Tool: get_insights
Level: ad
Fields: ad_id, ad_name, adset_id, campaign_id, campaign_name, spend, actions (purchase), cost_per_action_type (purchase), impressions, clicks, ctr, cpc
Date preset: {date_range}
```

### Step 2 — Classify every ad

For each ad, compute:
- **CPA** = cost per purchase (from cost_per_action_type). If zero purchases → CPA = ∞
- **Spend** = total spend in the period

Then classify:

```
IF spend < spend_threshold → UNDERTESTED
ELSE IF CPA < cpa_threshold → WINNER  
ELSE → LOSER
```

Ads with zero purchases AND spend ≥ spend_threshold are LOSERS (they spent enough and converted nothing).

### Step 3 — Present the classification

Before taking any action, show the user a summary table:

| Ad Name | Campaign | Spend | Purchases | CPA | Category |
|---|---|---|---|---|---|

Plus totals:
- X winners, Y undertested, Z losers
- Total spend analyzed
- Average CPA of winners

**Ask for confirmation before proceeding.**

### Step 4 — Create Scale campaign (Winners)

Only if there are winners:

1. **Create campaign**
   - Name: "{prefix}Scale - {date}"
   - Objective: OUTCOME_SALES
   - Status: PAUSED
   - Special ad categories: copy from original

2. **Create ad set**
   - Copy targeting, pixel, optimization from the original ad set of the first winner
   - Daily budget: {scale_budget}
   - Status: PAUSED

3. **For each winner ad:**
   - Get the ad's creative ID via `get_ad_details`
   - `duplicate_ad` into the new ad set
   - Status: PAUSED

### Step 5 — Create Retest campaign (Undertested)

Only if there are undertested ads:

1. **Create campaign**
   - Name: "{prefix}Retest - {date}"
   - Objective: OUTCOME_SALES
   - Status: PAUSED

2. **Create ad set**
   - Copy targeting, pixel, optimization from original
   - Daily budget: {retest_budget}
   - Status: PAUSED

3. **For each undertested ad:**
   - `duplicate_ad` into the new ad set
   - Status: PAUSED

### Step 6 — Pause Losers

For each loser ad:
- `update_ad` → status: PAUSED

If "keep original ads active" is false (default), also pause winners and undertested in their original campaigns after duplication.

### Step 7 — Summary report

Present final results:

```
✅ Scale campaign created: {campaign_id} — {N} winner ads, €{budget}/day
✅ Retest campaign created: {campaign_id} — {N} undertested ads, €{budget}/day  
⏸ {N} loser ads paused
⏸ {N} original ads paused (duplicated to new campaigns)

Everything is PAUSED — review and activate when ready.
```

## Edge cases

- **No winners found** → Skip scale campaign, tell the user
- **No undertested found** → Skip retest campaign, tell the user
- **All losers** → Only pause, suggest reviewing creative strategy
- **Ad has no purchases but very low spend** → Classify as undertested, not loser
- **Multiple ad sets in a campaign** → Pull insights at ad level across all ad sets, preserve original ad set settings when duplicating
- **Currency** → Use account currency, don't assume EUR

## Example

```
Account: act_123456789
CPA threshold: €25
Spend threshold: €10  
Date range: last_7d
Scale budget: €100/day
Retest budget: €50/day
```

Step 0 output:
```
Active campaigns in act_123456789 (last 7 days):

| # | Campaign | Spend | Purchases | CPA |
|---|---|---|---|---|
| 1 | Summer Sale - Static | €450 | 22 | €20.45 |
| 2 | Summer Sale - Dynamic | €380 | 18 | €21.11 |
| 3 | Retargeting - Catalog | €120 | 8 | €15.00 |
| 4 | Brand Awareness | €200 | 0 | - |

Which campaigns do you want me to analyze?
```

User: "1 and 3"

Then:
```
Found 47 ads across 2 selected campaigns:
- 12 Winners (avg CPA €14.30)
- 8 Undertested (avg spend €4.20)  
- 27 Losers (avg CPA €52.10)

Proceed? [Yes/No]
```
