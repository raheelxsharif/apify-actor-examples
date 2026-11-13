# Gartner Peer Insights Product & Market Data Extractor

Gartner Peer Insights is where enterprise IT buyers compare B2B software based on verified customer reviews. This actor pulls three kinds of data from it: a product's full rating profile and the reviews available to the public, every product listed under a Gartner market, or the complete directory of Gartner's markets and categories.

## What this actor does

- Fetches product profiles: name, vendor, markets, overall rating, star distribution, and per-dimension ratings (evaluation, integration, support)
- Fetches the reviews publicly available on a product's own page, with headline, rating, reviewer job title, industry, and company size
- Pulls likes/dislikes text from featured reviews when Gartner highlights them
- Lists every product under a given Gartner market, or the full directory of markets and categories
- No field is sent when Gartner's page doesn't have that data for the record

## Output: product record (`type: "product"`)

- `url` — the product page URL
- `productId`, `productName`, `productSeoName` — product identity
- `vendorId`, `vendorName`, `vendorSeoName` — the vendor behind the product
- `markets[]` — the Gartner market(s) this product is listed under, each with `id`, `name`, `seoName`
- `totalReviews`, `averageRating` — overall review count and rating
- `ratingsBreakdown` — count of reviews at each star level, 1 through 5
- `dimensionRatings[]` — per-dimension average rating and review count (e.g. evaluation & contracting, integration & deployment)
- `alternatives[]` — competing products Gartner lists as alternatives, with their own `name`, `seoName`, `averageRating`, `totalReviews`
- `scrapedAt`

## Output: review record (`type: "review"`)

- `reviewId`, `reviewUrl` — review identifier and its page on Gartner
- `productSeoName` — which product this review belongs to
- `headline`, `summary` — review title and body text
- `likes`, `dislikes` — pros/cons text, present only on reviews Gartner highlights as featured
- `rating` — the reviewer's star rating
- `reviewDate` — when the review was posted
- `jobTitle`, `industry`, `companySize` — reviewer's stated role, industry, and company revenue band
- `incentivized` — whether the reviewer was incentivized to write it (`Incentivized`, `Not incentivized`, or `Organic`)
- `source` — which page section the review was collected from
- `scrapedAt`

## Output: category record (`type: "category"`)

- `marketId`, `marketName`, `marketSeoName` — the Gartner market
- `categoryName`, `categorySeoName` — the parent category the market sits under
- `scrapedAt`

## Output: no-data notice (`type: "gartner_scraper_error"`)

Pushed instead of a normal record when `mode=market` finds no products in the given market, or when `mode=products` extracts no data at all for the requested URLs.

- `reason` — short code, e.g. `no_market_products`, `no_data`
- `message` — human-readable explanation
- `scrapedAt`

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `categories` | `products` (product profiles + reviews) / `market` (products in a market) / `categories` (full market/category directory) |
| `productUrls` | array | `[]` | Gartner product page URLs (mode=products) |
| `marketSeoName` | string | – | Gartner market URL slug, e.g. `sales-force-automation-platforms` (mode=market) |
| `useFilters` | boolean | `true` | Also fetch each product's rating/sort filter variants to collect more unique reviews |
| `maxItems` | integer | `100` | Maximum number of records to emit (1–10000) |
| `proxyConfiguration` | object | – | Apify proxy settings; residential is recommended for larger runs |

### Example: scrape specific products

```json
{
  "mode": "products",
  "productUrls": [
    "https://www.gartner.com/reviews/product/salesforce-sales-cloud",
    "https://www.gartner.com/reviews/product/hubspot-crm"
  ],
  "useFilters": true
}
```

### Example: list every product in a market

```json
{
  "mode": "market",
  "marketSeoName": "sales-force-automation-platforms"
}
```

### Example: list all markets and categories

```json
{
  "mode": "categories",
  "maxItems": 500
}
```

## Use cases

- **Vendor evaluation** — compare rating breakdowns and dimension scores before shortlisting software
- **Competitive intelligence** — monitor a competitor's rating trend and what reviewers call out as pros/cons
- **Market mapping** — list every product in a Gartner market to understand the competitive landscape
- **Sales enablement** — pull reviewer industry/company-size data to spot which segments like a product most
- **Content and research** — cite real reviewer feedback and rating distributions in analyst-style reports

## FAQ

**Does this require a Gartner account?**
No. It reads product, review and market pages that are publicly visible without logging in.

**Can I get every review ever written for a product?**
No. Gartner only exposes a limited set of reviews to visitors who aren't logged in. With `useFilters` enabled, the actor also fetches each product's rating and sort filter variants to pick up additional unique reviews beyond the first page — typically bringing the total into the dozens per product rather than just the handful shown on the base page — but this still isn't the full review count shown in `totalReviews`.

**Why do some reviews have `likes`/`dislikes` and others don't?**
Only reviews Gartner highlights as featured ("critical" or "favorable") carry separate likes/dislikes text; ordinary reviews only have `summary`.

**How do I find a market's SEO name?**
Open a Gartner market page and copy the slug from its URL — for `https://www.gartner.com/reviews/market/sales-force-automation-platforms`, the `marketSeoName` is `sales-force-automation-platforms`.

**Do I need a proxy?**
Not for small runs. A residential proxy is recommended if you're running many products or markets in one job to keep access reliable at scale.

**How fresh is the data?**
Every run reads the live page at request time, so ratings, review counts and listings reflect Gartner's current site.

**Is this affiliated with Gartner?**
No. This is an independent, third-party actor; it is not affiliated with or endorsed by Gartner, Inc.
