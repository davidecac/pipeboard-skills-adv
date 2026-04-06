---
name: account-audit
description: "SKILL __Account Audit__ — Pull all performance data from a Meta Ads account, structure it into navigable dashboards and queryable data files. Covers: account overview, campaign breakdown, audience analysis (age/gender/placement/time), creative performance (image vs video, copy angles, hook rate), and A+ Advantage+ settings audit. Use this skill whenever the user mentions: 'audit', 'account audit', 'analizza account', 'analisi account', 'account review', 'performance review', 'cosa non va nell account', 'diagnosi account', 'health check', 'audit Meta', 'audit Facebook Ads', 'analisi campagne', 'panoramica account', 'account overview', or any request to understand what's happening across an entire ad account."
---

# Account Audit

Pull all performance data from a Meta Ads account, organize it into dashboards for visual analysis and structured data files for follow-up interrogation with Claude.

## What it does

Takes an ad account and a date range, pulls everything, and produces:

1. **Interactive dashboard** — React with tabs for each analysis dimension
2. **Structured data pack** — CSV files saved to disk, one per dimension, ready to be loaded back into context for Q&A

The skill does NOT produce a diagnosis. The data is the deliverable. Diagnosis happens in the conversation after, when the user asks questions and Claude has the structured data to answer from.

## What I need from you

1. **Ad account ID** — e.g. `act_1240474954407657`
2. **Date range** — Period to analyze (e.g. "last_90d", "2025-09-01 to 2026-03-31")
3. **Comparison period** (optional) — For trend analysis (e.g. "compare Sep-Nov vs Dec-Feb")
4. **Purchase event name** (optional) — Default: "purchase". Some accounts use custom events.

I'll automatically look up the account's page, Instagram, and pixel.

## Workflow

### Step 0 — Account setup

```
Tool: get_account_info
→ Extract: account name, currency, timezone

Tool: get_account_pages
→ Extract: page_id, page_name

Tool: get_instagram_accounts
→ Extract: instagram_account_id

Tool: get_pixels
→ Extract: pixel_id
```

Save as `account_meta.json`.

### Step 1 — Account overview

```
Tool: get_insights
Level: account
Fields: spend, impressions, reach, clicks, ctr, cpc, actions, cost_per_action_type, purchase_roas
Date preset: {date_range}
Breakdowns: none (aggregate)
```

Also pull with time increment (daily or monthly depending on range length) for trend line.

**Output:** `01_account_overview.csv` — columns: date, spend, impressions, reach, clicks, ctr, cpc, purchases, cpa, roas

### Step 2 — Campaign breakdown

```
Tool: get_campaigns
→ All campaigns (ACTIVE + PAUSED)

Tool: get_insights
Level: campaign
Fields: campaign_id, campaign_name, objective, spend, impressions, clicks, ctr, actions, cost_per_action_type, purchase_roas
Date preset: {date_range}
```

**Output:** `02_campaigns.csv` — columns: campaign_id, campaign_name, objective, status, spend, impressions, clicks, ctr, purchases, cpa, roas

### Step 3 — Ad set breakdown

```
Tool: get_insights
Level: adset
Fields: adset_id, adset_name, campaign_id, campaign_name, spend, impressions, clicks, actions, cost_per_action_type, daily_budget, optimization_goal
Date preset: {date_range}
```

**Output:** `03_adsets.csv` — columns: adset_id, adset_name, campaign_id, campaign_name, spend, purchases, cpa, daily_budget, optimization_goal

### Step 4 — Audience breakdowns

Pull account-level insights with demographic breakdowns:

```
Tool: get_insights (×4 calls)
Level: account (or campaign if user wants per-campaign)
Date preset: {date_range}

Call A — Breakdown: age, gender
Call B — Breakdown: publisher_platform, platform_position  
Call C — Breakdown: hourly_stats_aggregated_by_advertiser_time_zone
Call D — Breakdown: (monthly or weekly time increment for trend)
```

**Output files:**
- `04a_age_gender.csv` — columns: age, gender, spend, impressions, clicks, purchases, cpa, roas
- `04b_placements.csv` — columns: platform, position, spend, impressions, clicks, purchases, cpa, roas
- `04c_hourly.csv` — columns: hour, spend, impressions, clicks, purchases, cpa, roas
- `04d_trend.csv` — columns: period, spend, impressions, clicks, purchases, cpa, roas

### Step 5 — Creative performance

```
Tool: get_ads
→ All ads in the account for the period

Tool: get_insights
Level: ad
Fields: ad_id, ad_name, adset_id, adset_name, campaign_id, campaign_name, spend, impressions, clicks, ctr, cpc, actions, cost_per_action_type, purchase_roas
Date preset: {date_range}

Tool: get_ad_creatives (for each ad, or bulk)
→ Extract: creative_id, image_hash/video_id, thumbnail_url, object_story_spec (title, body, description, call_to_action, link)
```

For each creative, extract and flatten:
- **Format**: image or video (infer from whether video_id is present)
- **Headline**: from object_story_spec → link_data → title
- **Primary text**: from object_story_spec → link_data → message
- **Description**: from object_story_spec → link_data → description  
- **CTA type**: from call_to_action → type
- **Creative ID**: to detect same creative used across multiple ads
- **A+ settings**: asset_feed_spec or degrees_of_freedom_spec → advantage_plus creative enhancements, product_extensions (Show Products) OPT_IN/OPT_OUT

**Copy angle tagging**: Group ads by primary text content. Assign a short label to each distinct copy angle (e.g. "outlet discount", "free shipping", "brand story"). Count how many unique angles exist per time period to detect copy diversity collapse.

**Output files:**
- `05a_ads_performance.csv` — columns: ad_id, ad_name, adset_id, adset_name, campaign_id, campaign_name, creative_id, format, spend, impressions, clicks, ctr, cpc, purchases, cpa, roas
- `05b_creatives_detail.csv` — columns: creative_id, ad_id, format, headline, primary_text, description, cta_type, advantage_plus_settings, product_extensions
- `05c_copy_angles.csv` — columns: copy_angle_label, primary_text_sample, ad_count, total_spend, purchases, cpa

### Step 6 — Save data pack

Save all CSV files to `/home/claude/audit_{account_id}/`:

```
audit_act_XXXXX/
├── account_meta.json
├── 01_account_overview.csv
├── 02_campaigns.csv
├── 03_adsets.csv
├── 04a_age_gender.csv
├── 04b_placements.csv
├── 04c_hourly.csv
├── 04d_trend.csv
├── 05a_ads_performance.csv
├── 05b_creatives_detail.csv
└── 05c_copy_angles.csv
```

These files stay on disk so Claude can reload them in follow-up questions.

### Step 7 — Build dashboard

Create a React dashboard (.jsx) with tabs:

**Tab 1 — Overview**
- Total spend, purchases, CPA, ROAS as KPI cards
- Trend line (spend + CPA over time)

**Tab 2 — Campaigns**
- Table: all campaigns sorted by spend, with purchases, CPA, ROAS
- Highlight: campaigns with zero purchases, campaigns with CPA > 2× account average

**Tab 3 — Audience**
- Age × Gender heatmap (CPA as color)
- Placement breakdown bar chart
- Hourly performance chart (ROAS or CPA by hour)

**Tab 4 — Creatives**
- Table: all ads sorted by spend, with format, CPA, creative_id
- Flag: same creative_id appearing in multiple campaigns (with CPA comparison)
- Format split: image vs video aggregate performance

**Tab 5 — Copy & A+**
- Copy angles table: angle label, ad count, spend, CPA
- Copy diversity over time (number of distinct angles per month)
- A+ settings breakdown: OPT_IN vs OPT_OUT counts and avg CPA per setting

Save dashboard to output directory.

### Step 8 — Interrogation menu

Present the dashboard, then show the user an actionable menu. Each option is a pre-built analysis with specific files to load and a delta-based presentation — no speculation, only measurable comparisons.

```
Dashboard ready. Data pack saved with {N} CSV files.

What do you want to dig into?

📊 PERFORMANCE
1. Where is my budget going? (spend distribution)
2. What changed between period A and period B?
3. Which campaigns have zero or near-zero conversions?

👥 AUDIENCE
4. Who is my best customer? (age/gender/placement)
5. When should I be spending? (hourly/daily patterns)

🎨 CREATIVE
6. Image vs video — which format converts better?
7. Am I reusing the same creatives too much?
8. How diverse is my copy? (angle analysis over time)

🔍 HEALTH CHECK
9. Saturation check
10. Testing velocity check
11. A+ settings impact check

Just give me the number.
```

---

## Interrogation templates

Each menu option has a defined template: which files to load, what to compute, how to present. Claude follows these templates exactly — no conclusions, only deltas and comparisons.

### 1. Where is my budget going?

**Files:** `02_campaigns.csv`
**Present:**
- Campaigns ranked by spend (descending)
- % of total spend per campaign
- For each campaign: spend, purchases, CPA
- Highlight: campaigns with >10% of total spend and zero purchases

### 2. What changed between period A and period B?

**Files:** `04d_trend.csv`, `05a_ads_performance.csv`, `05c_copy_angles.csv`, `05b_creatives_detail.csv`

Ask user for the two periods. Then compute and show side by side:

| Metric | Period A | Period B | Delta |
|---|---|---|---|
| Daily spend | | | |
| Number of active ads | | | |
| Number of distinct creatives | | | |
| Number of copy angles | | | |
| Frequency (if available) | | | |
| % budget on ads older than 14 days | | | |
| A+ OPT_IN vs OPT_OUT split | | | |
| CPA | | | |
| Purchases | | | |

**No "because" statements.** Present the table and let the user see what moved together.

### 3. Which campaigns have zero or near-zero conversions?

**Files:** `02_campaigns.csv`
**Present:**
- Filter: campaigns with spend > 0 and purchases ≤ 1
- Table: campaign name, spend, impressions, clicks, purchases
- Total spend wasted on zero-conversion campaigns

### 4. Who is my best customer?

**Files:** `04a_age_gender.csv`, `04b_placements.csv`

**Present:**
- Age × gender table sorted by CPA (lowest first)
- Include: spend, purchases, CPA, % of total purchases per segment
- Minimum threshold: only show segments with ≥ 3 purchases (below that, not statistically meaningful — flag it)
- Same for placements: sorted by CPA, with volume threshold

**Confidence note:** If total purchases < 50, add: "⚠️ Low volume — {N} total purchases. These patterns may not hold with more data."

### 5. When should I be spending?

**Files:** `04c_hourly.csv`

**Present:**
- Hourly chart: spend and CPA per hour
- Highlight: top 3 hours by CPA (lowest) and top 3 by spend
- Gap analysis: hours where CPA is low but spend is also low (potential opportunity)
- Hours where CPA is high and spend is high (potential waste)

**Confidence note:** Same volume threshold. If purchases per hour < 3, that hour's CPA is unreliable.

### 6. Image vs video performance

**Files:** `05a_ads_performance.csv`

**Present:**
- Aggregate: image total spend/purchases/CPA vs video total spend/purchases/CPA
- Top 5 ads by CPA for each format
- Volume context: "Images: {N} ads, {X} total spend. Video: {N} ads, {X} total spend."

**No "images work better" conclusion.** If one format has 3× the spend of the other, the comparison is skewed — flag it.

### 7. Am I reusing creatives too much?

**Files:** `05a_ads_performance.csv`, `05b_creatives_detail.csv`

**Present:**
- Find creative_ids that appear in multiple ads/campaigns
- For each duplicate: show CPA per ad/campaign where it appears
- Table: creative_id, number of ads using it, campaigns, CPA range (min-max)
- Age of each creative (first seen vs now, if time data available)

### 8. How diverse is my copy?

**Files:** `05c_copy_angles.csv`, `04d_trend.csv`

**Present:**
- Number of distinct copy angles per month/period
- Trend: is it increasing, stable, or decreasing?
- Table: angle label, ad count, spend, CPA
- % of total spend concentrated on the top 1 and top 3 angles

### 9. Saturation check

**Files:** `04d_trend.csv`, `05a_ads_performance.csv`

**Present a panel of signals, no conclusion:**

| Signal | Value | Trend |
|---|---|---|
| CPA (weekly) | current vs 4 weeks ago | ↑ ↓ → |
| Frequency (if available) | current vs 4 weeks ago | ↑ ↓ → |
| % spend on ads > 14 days old | | |
| Number of new creatives last 2 weeks | | |
| Same creative_id best CPA: first week vs now | | |

If ≥3 signals trend negative → "Multiple signals suggest potential saturation. Consider testing new creatives and audiences."

If signals are mixed → "No clear saturation pattern. Individual signals below for your assessment."

**This is the ONE place where Claude can suggest, but only with ≥3 converging signals, and framed as "consider" not "you have."**

### 10. Testing velocity check

**Files:** `05a_ads_performance.csv`, `04d_trend.csv`

**Present:**
- Number of new ads launched per week/month (based on first date with spend)
- % of total budget on ads with < 7 days of data
- Number of ads that received < spend_threshold (from winner-scaler) and were paused — "killed before tested"

No opinion on whether it's "enough." Just the numbers with comparison across periods if available.

### 11. A+ settings impact check

**Files:** `05b_creatives_detail.csv`, `05a_ads_performance.csv`

**Present:**
- Split ads by A+ setting (OPT_IN vs OPT_OUT) for each feature (enhancements, product_extensions)
- For each split: count of ads, total spend, avg CPA
- Table format, no recommendation

**Caveat always included:** "This is a correlation, not an A/B test. Ads with different A+ settings may also differ in creative, audience, and budget."

---

## Confidence framework

Every interrogation template follows these rules:

1. **Volume thresholds**: Any segment with < 3 conversions gets flagged as "⚠️ Low volume — treat with caution"
2. **Spend imbalance**: If comparing two groups and one has 3×+ the spend, flag: "⚠️ Uneven spend — {X}€ vs {Y}€. Comparison may be skewed."
3. **Correlation ≠ causation**: Any time two metrics move together, never say one caused the other. Say "these moved together" and list both.
4. **No superlatives without context**: Never say "best" or "worst" without showing the volume. "Lowest CPA" always comes with "on {N} purchases and {X}€ spend."

## Important notes

- All data is pulled via Pipeboard Meta MCP tools, not Ads Manager exports
- Currency and timezone come from the account settings, never assume EUR
- The dashboard hardcodes data from the CSVs — if the user wants to refresh, re-run the skill
- For accounts with 100+ ads, the creative detail step may require multiple API calls — batch where possible using `bulk_get_ad_creatives` and `bulk_get_insights`
- The skill is read-only except for pulling data — it never creates, modifies, or pauses anything

## Example

```
Account: act_1240474954407657
Date range: September 1, 2025 to March 31, 2026
Compare: Sep-Nov vs Dec-Feb vs Mar
```

Output: 5-tab dashboard + 10 CSV files covering 6 months of performance data, ready for interrogation.
