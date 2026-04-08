# Single Product Campaign Manager

Manage Meta Ads campaigns where each product gets its own dedicated ad. Add new products, pause underperformers, redistribute ads across ad sets — all from Claude Code with Pipeboard.

## Who is this for

Do you run an e-commerce store and advertise your products on Meta Ads? You probably have (or want) campaigns where each product has its own ad: one photo, the product name, the price, and a direct link to the product page. This type of campaign gives you full control — you know exactly which products sell and which don't, you can kill the losers and add new arrivals fast.

This skill automates the entire process: from adding new products to distributing them across campaigns, to pausing ads that aren't converting.

## How single product campaigns work

These are Meta Ads campaigns (Sales objective) where each ad promotes a single product with:
- A product photo
- Price + product name as headline (e.g. "€34.90 - Sophie Dress")
- A generic brand copy as primary text
- A short description (e.g. "Free shipping 24/48h")
- SHOP_NOW CTA linking directly to the product page

Unlike dynamic catalog campaigns, each ad is created manually. The advantage is total control: you see exactly which product performs, you can pause the flops and add new arrivals immediately.

Typically you create multiple campaigns of this type, each with multiple ad sets, keeping a maximum of N ads per ad set to avoid diluting the budget too much.

## What you can do

- **Add products** — provide your product URLs, everything else is automatic (images, name, price extracted from the page)
- **Pause ads** — send a screenshot from Business Manager with the ads highlighted, or ask for a performance recap and decide
- **Check performance** — recap by campaign, ad set, or individual ad with CPA, ROAS, spend
- **Redistribute** — rebalance ads across ad sets when needed

## Getting started

1. Make sure you have [Pipeboard](https://pipeboard.co) configured as MCP in Claude Code
2. Copy the `single-product-campaign` folder to your computer
3. Open Claude Code in that folder
4. Type: **"Let's set up my account"**

Claude will guide you through a questionnaire to configure everything. After setup, you're operational.

## Requirements

- Pipeboard account with access to your Meta Ads account
- An e-commerce website with accessible product pages (Shopify, WooCommerce, etc.)
- At least one active Meta Ads campaign with Sales objective (or we'll create one together)
