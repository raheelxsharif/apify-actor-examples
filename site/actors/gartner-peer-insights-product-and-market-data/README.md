# Gartner Peer Insights Product & Market Data Extractor

Gartner Peer Insights is where enterprise buyers compare B2B software through verified customer reviews. This actor collects four connected layers of that data without a login: product profiles, the public review stream with reviewer demographics, complete product listings for a Gartner market, and the full market/category directory.

## Modes

| Mode | What it returns | Best for |
|---|---|---|
| `products` | One `product` record plus `review` records for each product URL or slug | Vendor and product research |
| `market` | One `market` record plus every product in that market, or full product details when `includeProductDetails` is on | Competitive landscape and market mapping |
| `categories` | One `category` record per Gartner market, with its parent category | Building a market taxonomy |

Product mode accepts full URLs or bare slugs, and market mode accepts a slug or a full market URL.

## Input

| Field | Type | Default | What it does |
|---|---|---|---|
| `mode` | select | `categories` | `products`, `market`, or `categories` |
| `productUrls` | string list | – | Product page URLs or slugs for `products` |
| `marketSeoName` | string | – | Market slug or URL for `market` |
| `includeProductDetails` | boolean | `false` | In `market` mode, fetch full profiles and reviews instead of fast listing rows |
| `maxReviewsPerProduct` | integer | `50` | Public reviews per product; `0` = profile only |
| `useFilters` | boolean | `true` | Adds one pass per star rating after paging to surface extra unique reviews |
| `reviewSort` | select | `most-helpful` | `most-helpful`, `most-recent`, `least-recent` |
| `reviewRating` | select | `all` | `all`, `5`, `4`, `3`, `2`, `1` |
| `reviewerType` | select | `all` | `all`, `end-user`, `partner` |
| `lastTwelveMonths` | boolean | `false` | Limit reviews to the last 12 months |
| `marketSort` | select | `ratingsCount_desc` | Ratings count, average rating, or name; ascending or descending |
| `marketCompanySize` | select | `all` | Gartner company-size facet for market listings |
| `maxItems` | integer | `100` | Total record cap across all output types |
| `proxyConfiguration` | object | – | Optional; the actor tries direct access first |

### Examples

```json
{
  "mode": "products",
  "productUrls": [
    "https://www.gartner.com/reviews/product/salesforce-sales-performance-management",
    "microsoft-dynamics-365-sales"
  ],
  "maxReviewsPerProduct": 25,
  "reviewSort": "most-recent"
}
```

```json
{
  "mode": "market",
  "marketSeoName": "sales-force-automation-platforms",
  "marketSort": "averageRating_desc",
  "marketCompanySize": "50M-1B USD",
  "maxItems": 100
}
```

```json
{
  "mode": "market",
  "marketSeoName": "sales-force-automation-platforms",
  "includeProductDetails": true,
  "maxReviewsPerProduct": 10,
  "maxItems": 50
}
```

```json
{
  "mode": "categories",
  "maxItems": 500
}
```

## Output: product (`type: "product"`)

- `url`, `productId`, `productName`, `productSeoName`, `productLogoUrl`, `productLegacy`
- `vendorId`, `vendorName`, `vendorSeoName`, `vendorDescription`, `vendorLogoUrl`, `vendorClaimedStatus`
- `markets[]` — each with `id`, `name`, `seoName`, overview text, last-updated date, and awards
- `productDescription`, `pricingDescription`, `productOverviewUpdatedAt`
- `totalReviews`, `averageRating`, `ratingsBreakdown` (counts for 1–5 stars), `dimensionRatings[]`
- `alternatives[]` — competing products with `name`, `seoName`, `averageRating`, `totalReviews`, `logoUrl`
- `scrapedAt`

Market listing rows use the same `product` type but are leaner: `url`, `productId`, `productName`, `productSeoName`, `vendorName`, `vendors[]`, `marketSeoName`, `averageRating`, `totalReviews`, `productDescription`, `hasProfile`, `legacy`, `scrapedAt`.

## Output: review (`type: "review"`)

- `reviewId`, `reviewUrl`, `productSeoName`, `productNames`
- `headline`, `summary`, `likes`, `dislikes` (likes/dislikes only when Gartner features the review)
- `rating`, `reviewDate`, `reviewDateIso`
- `jobTitle`, `industry`, `companySize`, `reviewerType`
- `incentivized`, `reviewOrigin`, `incentive` — Gartner's review-origin and gift disclosures
- `source` — which section the review came from, `scrapedAt`

## Output: market (`type: "market"`)

`marketId`, `marketName`, `marketSeoName`, `marketUrl`, `marketTitle`, `marketSubtitle`, `marketDescription`, `marketReviewCount`, `productCount`, `scrapedAt`.

## Output: category (`type: "category"`)

`marketId`, `marketName`, `marketSeoName`, `marketUrl`, `isPopularMarket`, `categoryId`, `categoryName`, `categorySeoName`, `categoryDescription`, `categoryShortDescription`, `scrapedAt`.

## Pagination and depth

- Market listings are fetched in pages of 100 until Gartner's reported product count is reached.
- Reviews are paged one result page at a time. For logged-out visitors each result page exposes one full public review, so the actor pages until `maxReviewsPerProduct` or the public list ends. `useFilters` adds one extra query per star rating to surface additional unique reviews.
- Gartner's `totalReviews` counter includes reviews that are not publicly readable. The dataset therefore contains fewer reviews than the counter by design; this is a source limit, not an actor failure.
- Review text is never fabricated. Gartner's placeholder summaries are omitted.

## FAQ

**Does this need a Gartner account or cookies?**
No. It reads the public product, review, market, and category views anonymously.

**Do I need a proxy?**
No. Direct requests are attempted first. If Gartner blocks the network path, the actor escalates automatically; you do not need to configure a proxy.

**Can I get every review ever written?**
No. Gartner exposes only part of each product's review stream to logged-out visitors. The actor collects everything that stream exposes through paging and rating passes up to your cap.

**Why is `totalReviews` larger than the number of review rows?**
`totalReviews` is Gartner's full rating count. Public review pages expose one review at a time per result page, so a run with `maxReviewsPerProduct: 50` collects at most 50 public reviews unless additional star-rating passes find new ones.

**How do I find a market slug?**
Open the market page and copy the last URL segment: `https://www.gartner.com/reviews/market/sales-force-automation-platforms` → `sales-force-automation-platforms`. Full URLs are also accepted.

**Are the review and product URLs public?**
Yes. Review URLs use Gartner's `/reviews/review/view/<id>` path, and product and market URLs use Gartner's public Peer Insights paths.

**Is this affiliated with Gartner?**
No. This is an independent third-party actor and is not endorsed by Gartner, Inc.
