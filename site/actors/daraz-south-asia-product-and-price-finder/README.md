# Daraz South Asia Product & Price Finder

Search **Daraz** — South Asia's largest online marketplace, covering Pakistan (daraz.pk), Bangladesh (daraz.com.bd), Sri Lanka (daraz.lk), and Nepal (daraz.com.np). Search products by keyword or browse Daraz's real category taxonomy (Mobiles & Tablets, Fashion, Health & Beauty, Groceries, and more), then filter by price range, star rating, and sort order. No login or cookies required.

## What this actor does

- **Two modes:** `search` (keyword) and `category` (browse Daraz's real navigation categories)
- **Four storefronts:** Pakistan, Bangladesh, Sri Lanka, Nepal — pick per run
- **Filters:** min/max price, minimum star rating, sort order (popularity / price low-high / price high-low), brand, shipped-from region, free-shipping only, in-stock only
- **Rich product data:** price, discount %, rating, review count, units sold, seller, shipping origin, and product images
- **Empty fields are omitted**

## Output per product

- `productId` — Daraz's internal item ID
- `title` — product listing title
- `price`, `priceFormatted` — numeric price and the storefront-formatted string (e.g. `Rs. 116,000`)
- `originalPrice`, `discountPercentage` — only present when the item is on sale
- `currency` — ISO code for the storefront (`PKR` / `BDT` / `LKR` / `NPR`)
- `rating`, `reviewCount` — only present when the product has reviews
- `itemsSoldText`, `itemsSold` — units sold, as shown and parsed to a number
- `brand`, `brandId`
- `sellerName`, `sellerId`
- `shippedFrom` — origin province/region as shown on the listing
- `imageUrl` — primary product image
- `productUrl` — canonical product page link
- `inStock`, `isSponsored`, `freeShipping`
- `categoryIds` — Daraz's internal category IDs for the product
- `countrySite` — which storefront the record came from (`pk` / `bd` / `lk` / `np`)
- `recordType: "product"`, `scrapedAt`

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `search` | `search` (keyword) or `category` (browse) |
| `countrySite` | string | `pk` | `pk` / `bd` / `lk` / `np` — which Daraz storefront |
| `searchQuery` | string | `mobile phone` | Free-text keyword (mode=search; also narrows results within `mode=category`) |
| `category` | string | `phones-tablets` | Category to browse (mode=category) — see the dropdown for the full list |
| `brand` | string | – | Only keep products whose brand exactly matches this name (case-insensitive), e.g. `Samsung` |
| `shippedFrom` | string | – | Only keep products shipped from this region (case-insensitive exact match), e.g. `Punjab`, `Dhaka` |
| `freeShippingOnly` | boolean | `false` | Only keep products with a free-shipping badge |
| `inStockOnly` | boolean | `false` | Only keep products Daraz currently reports as in stock |
| `minPrice` | int | – | Drop products priced below this amount (local currency) |
| `maxPrice` | int | – | Drop products priced above this amount (local currency) |
| `minRating` | string | `0` | `0` (any) to `5` (5 stars only) |
| `sortBy` | string | `popularity` | `popularity` / `priceasc` / `pricedesc` |
| `maxItems` | int | `50` | Hard cap on emitted records (1–2000) |
| `proxyConfiguration` | object | `{"useApifyProxy": true}` | Apify proxy config (AUTO datacenter group is sufficient) |

### Example: keyword search with price range

```json
{
  "mode": "search",
  "countrySite": "pk",
  "searchQuery": "wireless earbuds",
  "minPrice": 1000,
  "maxPrice": 5000,
  "sortBy": "priceasc",
  "maxItems": 50
}
```

### Example: browse a category, highly rated only

```json
{
  "mode": "category",
  "countrySite": "bd",
  "category": "beauty-health",
  "minRating": "4",
  "sortBy": "popularity",
  "maxItems": 100
}
```

### Example: Nepal storefront, price high to low

```json
{
  "mode": "category",
  "countrySite": "np",
  "category": "computing",
  "sortBy": "pricedesc",
  "maxItems": 30
}
```

### Example: Sri Lanka search, narrow by keyword within a category

```json
{
  "mode": "category",
  "countrySite": "lk",
  "category": "sports-travel",
  "searchQuery": "badminton",
  "maxItems": 40
}
```

## Use cases

- **Price monitoring** — track pricing and discounts on specific products or categories across South Asian markets
- **Market research** — compare product assortments and pricing between Pakistan, Bangladesh, Sri Lanka, and Nepal storefronts
- **Competitor analysis** — monitor seller listings, ratings, and units-sold signals for a category
- **Deal discovery** — filter by discount and rating to surface the best-value listings
- **Catalog enrichment** — pull structured product data (images, brand, seller) for downstream cataloging

## Data Source / Limitations

- Data comes from Daraz's own product-listing endpoint (the same one that powers the on-site search and category pages) — no scraping of unofficial mirrors or third-party resellers.
- Daraz's public search results page (`/catalog/`) is excluded by the site's `robots.txt`; this actor accesses it directly like the many third-party Daraz scrapers already on the market, and does not bypass any login, paywall, or CAPTCHA.
- `sortBy` only exposes the three sort orders Daraz's own UI provides (`popularity`, `priceasc`, `pricedesc`) — Daraz does not offer a distinct "newest" sort on product listings, so it is not included.
- `minRating` filters to Daraz's own rating buckets. Because most listings for a given search cluster at 4+ stars, low thresholds (1–3) and "any rating" often return very similar result sets — this reflects Daraz's actual rating distribution, not a bug.
- Category names and slugs reflect Daraz's real, currently-live navigation taxonomy as of the last verification pass; Daraz occasionally renames or merges categories.
- `brand` and `shippedFrom` are exact-match (case-insensitive) text filters rather than dropdowns — Daraz's brand and shipping-region lists differ per category/storefront and aren't a fixed, query-independent set, so they can't be enumerated reliably.

## FAQ

**Is this affiliated with Daraz?**  No. This is an independent, third-party actor that reads Daraz's public product-listing data. It is not endorsed by or affiliated with Daraz or Alibaba Group.

**Do I need a Daraz account, login, or API key?**  No. All modes work without authentication.

**Which countries are supported?**  Pakistan (`daraz.pk`), Bangladesh (`daraz.com.bd`), Sri Lanka (`daraz.lk`), and Nepal (`daraz.com.np`) — pick via `countrySite`.

**Can I combine keyword search with a category?**  Yes — set `mode` to `category` and also fill in `searchQuery` to narrow results to that keyword within the chosen category.

**Why do some products have no `rating` or `reviewCount`?**  Those fields are omitted when a product has no reviews yet — Daraz doesn't show a rating for unreviewed listings.

**Why is `originalPrice` sometimes missing?**  It's only included when the product is actively discounted. Full-price listings only have `price`.

**How fresh is the data?**  Real-time — every run queries Daraz's live listing endpoint directly.

**What does `itemsSoldText` vs `itemsSold` mean?**  `itemsSoldText` is Daraz's own display string (e.g. `"16.1K sold"`); `itemsSold` is that same value parsed into a plain number (`16100`) for easier filtering and sorting.
