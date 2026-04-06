---
name: single-product-campaign
description: "SKILL __Single Product Campaign at Scale__ — Launch a full Meta Ads campaign with individual ads for each product. Create 100+ ads in minutes instead of hours — just provide product URLs and images. Use this skill whenever the user mentions: 'campagna prodotti', 'single product ads', 'un ad per prodotto', 'lancia prodotti', 'crea ads da URL', 'product campaign', 'catalogo manuale', 'ads from product pages', 'bulk ads', 'crea 50 ads', 'crea 100 ads', 'ogni prodotto un ad', or any request to create multiple ads from a list of product URLs with images."
---

# Single Product Campaign at Scale

Launch a full Meta Ads campaign with individual ads for each product. Create 100+ ads in minutes instead of hours — just provide product URLs and images.

## When to use

You have a list of products and want to create a campaign where each product gets its own ad with:

- Single product image
- Price + product name as headline (extracted automatically from the product page)
- Generic brand copy as primary text
- Custom description (e.g. "Free shipping 24/48h")
- SHOP_NOW CTA
- All Advantage+ creative features enabled (including Show Products / product_extensions)
- Pixel tracking for purchases

## What I need from you

1. **Account**: Which ad account to use (I'll look up page, Instagram, and pixel automatically)
2. **Products**: For each product provide:
   - Product page URL
   - Image (local file or URL)
3. **Primary text**: The generic ad copy (same for all ads)
4. **Description**: Text shown below the headline (e.g. "Free shipping 24/48h")
5. **Targeting**: Country, gender, age range
6. **Daily budget**: In account currency
7. **Product set ID**: The catalog product set for Show Products (I can help you find it)

## What I'll do

1. Fetch account details (page_id, instagram_id, pixel_id)
2. Visit each product URL to extract product name and price automatically
3. Upload all product images to Meta in parallel
4. Create one OUTCOME_SALES campaign (Lowest Cost, Auction)
5. Create one ad set with your targeting, pixel tracking (Purchase), product_set_id, and DSA compliance
6. For each product, create a creative with:
   - Your product image
   - Headline: "€{price} - {product_name}" (auto-extracted)
   - All Advantage+ features enabled including product_extensions (Show Products)
7. Create one ad per product with pixel tracking
8. Everything is created in PAUSED status — you review and activate when ready

## Example

```
Account: act_123456789
Primary text: "Discover our outlet — up to 80% off. Dress well, pay less."
Description: "Free shipping 24/48h"
Targeting: Italy, men, 18-65
Budget: €70/day
Product set: 654849950452874
Products:
1. https://mystore.com/product-1 → image1.jpg
2. https://mystore.com/product-2 → image2.jpg
3. https://mystore.com/product-3 → image3.jpg
```

I'll automatically extract each product's name and price from the URLs, upload the images, and create all ads.
