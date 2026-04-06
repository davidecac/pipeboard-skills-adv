# Pipeboard Skills

Ready-to-use Claude skills for Meta Ads management via [Pipeboard](https://pipeboard.co) MCP.

Each skill turns a multi-step media buying workflow into a single conversation. Install a skill, describe what you need, and Claude handles the rest — campaign creation, performance analysis, creative scaling — all through natural language.

## Requirements

- [Claude Pro/Team/Enterprise](https://claude.ai) with skills support
- [Pipeboard Meta MCP](https://pipeboard.co) connected to your Claude workspace
- A Meta Ads account with active campaigns (for audit and winner scaler)

## Skills

### 🛍️ Single Product Campaign

**Create 100+ ads in minutes instead of hours.**

Give Claude a list of product URLs and images. It visits each page, extracts product name and price, uploads images to Meta, and builds a full campaign with one ad per product — all in PAUSED status, ready for review.

Best for: e-commerce brands launching product-level ads at scale without a catalog.

→ [View skill](skills/single-product-campaign/SKILL.md)

---

### 🏆 Winner Scaler

**Analyze campaigns, scale winners, retest undertested, pause losers.**

Claude pulls performance data from your selected campaigns, classifies every ad by CPA and spend thresholds, then takes action:

| Category | Rule | Action |
|---|---|---|
| Winners | CPA below threshold | → New scale campaign |
| Undertested | Spend below threshold | → New retest campaign |
| Losers | High CPA, enough spend | → Paused |

Starts with a campaign overview so you pick which ones to analyze — not all campaigns are suitable (e.g. dynamic creative campaigns).

Best for: weekly creative review and budget reallocation.

→ [View skill](skills/winner-scaler/SKILL.md)

---

### 🔍 Account Audit

**Full account analysis with structured data you can query.**

Pulls everything from your account and organizes it into:

1. **Interactive dashboard** (React) with 5 tabs: overview, campaigns, audience, creatives, copy & A+
2. **10 CSV files** saved to disk for follow-up analysis

After the audit, Claude presents a menu of 11 pre-built analyses — each with defined data sources, delta-based presentation (no speculation), and confidence warnings for low-volume segments.

```
📊 Where is my budget going?
👥 Who is my best customer?
🎨 Image vs video — which converts better?
🔍 Saturation check
...and 7 more
```

Every answer shows comparisons and measurable deltas, never unsupported conclusions.

Best for: onboarding a new client, monthly performance review, pre-pitch account analysis.

→ [View skill](skills/account-audit/SKILL.md)

## Installation

1. Download the `.skill` file from [Releases](../../releases) (or zip the skill folder yourself)
2. In Claude.ai, go to **Settings → Skills**
3. Click **Add Skill** and upload the `.skill` file
4. Make sure Pipeboard Meta MCP is connected in your tools

Each `.skill` file is a zip containing the `SKILL.md` — you can also create it manually:

```bash
cd skills/winner-scaler
zip winner-scaler.skill SKILL.md
```

## How these skills work

Skills are instruction files that tell Claude how to handle specific workflows. When you describe a task that matches a skill's trigger phrases, Claude loads the skill and follows its steps — calling Pipeboard MCP tools to interact with your Meta Ads account.

Everything is created in **PAUSED** status. You always review before anything goes live.

## Contributing

Have a workflow you repeat every week? It's probably a skill. Open an issue or PR.

## License

MIT
