# Bed Bath & Beyond Price & Rewards Scraper

Collect live product data from the current [bedbathandbeyond.com](https://www.bedbathandbeyond.com) catalog. Search by keyword, browse a home-goods department, look up specific products, or pull full customer reviews — with prices, sale prices, ratings, review text, images, shipping flags and Welcome Rewards cash-back figures.

## What this actor does

- **Search products** by free-text keyword across the live catalog
- **Browse 17 home-goods departments** — bedding, bath, kitchen, furniture, decor, mattresses and more
- **Look up products** from product-page URLs or numeric product IDs
- **Collect customer reviews** with review text, star rating, author, date, verified-purchase flag and customer photos
- Filter products by price range, average rating, on-sale status, two-day shipping and brand/title text
- Filter reviews by star rating, verified buyers and reviews with photos
- Return only fields that have a real value — nothing is emitted as a placeholder

## Output

Every record carries `recordType` (`product` or `review`) and `scrapedAt` (UTC ISO timestamp).

### Product records

- `productId`, `sku` — catalog identifiers
- `title`, `brand`, `productUrl`, `sourceUrl`
- `price`, `priceMax`, `currency` — current price; `priceMax` is set when size/color options have different prices
- `originalPrice`, `discountPercent`, `onSale`, `saleEndDate`
- `rating`, `reviewCount`, `ratingCount`, `ratingHistogram`, `recommendedRatio`
- `description`, `shortDescription`, `countryOfOrigin`, `firstListed`, `inStock`, `freeShipping`
- `imageUrl`, `fullImageUrl`, `breadcrumbs`, `specifications`, `optionCount`
- `urgencyMessaging`, `valueMessaging`, `twoDayShipping`, `isSponsored`
- `rewardsAmount`, `rewardsPoints`, `rewardsPercentage`
- `category` (department mode) or `searchKeywords` (keyword search)

Product-detail fields beyond the search card (`description`, `specifications`, etc.) are populated when the site's product API is reachable; otherwise the actor falls back to the search card plus the review rollup.

### Review records

- `reviewId`, `legacyReviewId`, `productId`, `productName`, `productUrl`
- `title`, `text`, `rating`, `author`, `location`
- `createdDate`, `updatedDate`
- `verifiedBuyer`, `verifiedReviewer`, `staffReviewer`
- `helpfulVotes`, `notHelpfulVotes`, `helpfulScore`
- `variantId`, `upc`, `gtin`, `locale`, `images`

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `search` | `search`, `byCategory`, `product` or `reviews` |
| `keywords` | string | — | Free-text query for `search`, e.g. `towel`, `comforter set` |
| `category` | string | `Bedding` | Department for `byCategory` (17 options) |
| `productUrls` | array | — | Product-page URLs or numeric IDs for `product` / `reviews` |
| `sortBy` | string | `relevance` | Product sort: `relevance`, `onSale`, `bestSelling`, `priceLowHigh`, `priceHighLow`, `rating`, `newArrivals` |
| `minPrice` | integer | — | Drop products below this price (server-side filter) |
| `maxPrice` | integer | — | Drop products above this price (server-side filter) |
| `minRating` | integer | — | Minimum product rating, or minimum review star rating |
| `maxRating` | integer | — | Maximum product rating, or maximum review star rating |
| `onSaleOnly` | boolean | `false` | Only on-sale products |
| `fastShippingOnly` | boolean | `false` | Only two-day-shipping products |
| `brandContains` | string | — | Keep products whose brand or title contains this text |
| `maxItems` | integer | `30` | Max product records for search/department/product modes |
| `reviewSort` | string | `newest` | `newest`, `oldest`, `highestRating`, `lowestRating`, `mostHelpful` |
| `maxReviews` | integer | `50` | Max review records across all product references |
| `verifiedOnly` | boolean | `false` | Only verified-purchase reviews |
| `withImagesOnly` | boolean | `false` | Only reviews that include customer photos |

### Examples

Keyword search:

```json
{
  "mode": "search",
  "keywords": "comforter set",
  "maxItems": 30
}
```

Browse a department, on sale and highly rated:

```json
{
  "mode": "byCategory",
  "category": "Bath",
  "onSaleOnly": true,
  "minRating": 4,
  "maxItems": 40
}
```

Product lookups from URLs or IDs:

```json
{
  "mode": "product",
  "productUrls": [
    "https://www.bedbathandbeyond.com/Bedding-Bath/Superior-Egyptian-Cotton-900-GSM-Luxurious-Absorbent-6-piece-Towel-Set/5296998/product.html",
    "44226118"
  ]
}
```

Recent five-star reviews with photos:

```json
{
  "mode": "reviews",
  "productUrls": ["5296998"],
  "reviewSort": "newest",
  "minRating": 5,
  "withImagesOnly": true,
  "maxReviews": 50
}
```

## Use cases

- **Price monitoring** — track catalog prices, sale prices and discount percentages over time
- **Assortment research** — see what is on sale, best-selling or newly listed in a department
- **Competitive analysis** — compare home-goods pricing and ratings across retailers
- **Voice-of-customer analysis** — collect review text and star ratings for sentiment and topic research
- **Affiliate and content sites** — source product data, images and ratings for buying guides
- **Deal aggregation** — surface highly rated, on-sale products across departments

## Data source & limitations

- **Catalog search** — the site's server-rendered `/search` page embeds a Next.js `__NEXT_DATA__` payload with product cards, price, sale, rating, review count, images, shipping and rewards fields. No login or cookies are needed.
- **Price filter** — `minPrice` / `maxPrice` are sent to the catalog as its own `price=<min>:<max>` filter, so the fetched window is already price-filtered.
- **Other product filters** — the server ignores rating, brand and promotion query parameters, so `minRating`, `maxRating`, `onSaleOnly`, `fastShippingOnly` and `brandContains` are applied to the fetched records.
- **Sorting** — the site's sort links are client-side only and are ignored by a plain HTTP request, so product sorting is applied to the fetched result window. For a true lowest/highest price across a large department, raise `maxItems` and pair it with a price sort.
- **Direct department pages** — the site's `/c/...` department pages are bot-protected. Department mode reaches each department through the catalog's own search relevance for the department name; the department is recorded in `category`.
- **Product details** — product-page HTML and `/api/product.json` are bot-protected from most network exits. The actor tries the product API first, then falls back to the search card, the product URL exposed by the search redirect, and the review rollup. Core price/rating/review-count data is therefore always available for a valid product ID or URL.
- **Reviews** — full review text comes from the site's public PowerReviews display endpoint. Reviews are paginated 25 at a time; `maxReviews` caps the run.
- **Currency** — USD only (US storefront).
- **Images** — CDN image URLs are unsigned and do not expire.
- **No affiliation** — this is an independent third-party actor, not affiliated with or endorsed by Bed Bath & Beyond or its operating company.

## FAQ

**Do I need a proxy or login?**
No. The actor works with no account, cookies or proxy setup.

**Will my old saved task keep working?**
Yes. Legacy input keys are mapped in code: `query`, `searchQuery` and `q` become `keywords`; `department` and `categoryName` become `category`; `productId`, `productUrl`, `urls` and `startUrls` become `productUrls`; snake_case variants such as `max_items` and `min_price` are also accepted. Old tasks do not need to be edited.

**Why is `brand` sometimes missing?**
Search cards do not carry an explicit brand field. The actor matches the title against the catalog's own brand facet for the query; generic or unbranded listings are returned without `brand`.

**Why do some reviews have no photos?**
Only reviews that include customer images carry an `images` array. Use `withImagesOnly` to keep only those.

**Can I get reviews for several products at once?**
Yes. Put multiple URLs or IDs in `productUrls`. `maxReviews` is the total cap across all of them.

**Is product-detail data always complete?**
Core pricing and review data is. Extended detail fields such as `description` and `specifications` depend on the product API being reachable from the run's network exit; when it is not, the record still carries the search-card price and the review rollup.
