# Myntra Fashion Catalog & Review Scraper

Scrape **Myntra** fashion listings and product pages for pricing, discounts, coupons, sellers, size-level stock, specifications, and written reviews. Start from a keyword search, a category, a brand, a listing URL, or a list of product IDs, then export clean records that are ready for pricing research, catalog monitoring, or review analysis.

## What this actor does

- **Five input modes:** keyword search, category browse, brand browse, listing/product URLs, and numeric product IDs
- **Multiple inputs per run:** several queries, categories, brands, URLs, or IDs are fetched in one job and deduplicated by product
- **Real pagination:** walks past the first listing page and splits large searches by gender, price band, and sort order
- **Server-side filters:** gender, sort order, and price range are applied by Myntra before records are collected; discount, rating, and name keywords are checked on each record
- **Full product enrichment:** sellers and addresses, per-size stock and garment measurements, specifications, rating breakdown, return/COD policies, videos, tags, and coupons
- **Optional review rows:** written reviews with rating, text, size bought, photos, votes, and dates
- **Empty fields are omitted** — records never contain `null`, `""`, `[]`, or `{}`

## Output per product

Product rows have `recordType: "product"`.

- `productId`, `productName`, `brand`, `additionalInfo`
- `currency` — always `INR`
- `mrp`, `price`, `discountAmount`, `discountPercent`, `discountDisplayLabel`
- `couponCode`, `couponDiscount`, `priceWithCoupon`, `couponValidUntil`
- `rating`, `ratingCount`, `reviewsCount`, `reviewCount`, `ratingDistribution`
- `primaryColour`, `baseColour`, `gender`, `season`, `year`
- `sizes[]`, `availableSizes[]`, `sizeOptions[]` — per-size SKU, availability, stock, price, sellers, and measurements
- `hasMultipleSizes`, `inStock`, `totalStock`, `colourVariantImages[]`, `colours[]`, `relatedColoursCount`
- `sellerId`, `sellerName`, `sellerCity`, `sellerState`, `sellers[]` — partner ID, name, display name, and full address
- `description`, `materialCare`, `sizeFit`, `manufacturer`, `countryOfOrigin`
- `specifications` — garment attributes such as fabric, fit, pattern, sleeve, occasion, and wash care
- `tags[]`, `offers[]`, `videoUrls[]`, `brandLogoUrl`
- `imageUrl`, `imageUrls[]`
- `category`, `subCategory`, `masterCategory`, `categoryPath`, `listedAt`
- `isReturnable`, `isExchangeable`, `codAvailable`, `returnDays`, `tryAndBuy`
- `productUrl`, `sourceUrl`, `query`, `sourceKind`, `detailsFetched`, `position`, `scrapedAt`

## Output per review

Review rows have `recordType: "review"` and are saved only when `includeReviews` is enabled.

- `reviewId`, `productId`, `reviewedProductId`
- `rating`, `text`, `sizeBought`
- `images[]`, `videoUrl`
- `upvotes`, `downvotes`
- `reviewerName`, `reviewedAt`
- `productName`, `brand`, `productUrl`, `sourceUrl`, `scrapedAt`

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `search` | `search` / `browseByCategory` / `browseByBrand` / `byUrl` / `byProductIds` |
| `searchQueries` | array | `[]` | Keyword searches (mode=search) |
| `categories` | array | `[]` | Category listings (mode=browseByCategory) |
| `brands` | array | `[]` | Brand listings (mode=browseByBrand) |
| `startUrls` | array | `[]` | Myntra listing or product URLs (mode=byUrl) |
| `productIds` | array | `[]` | Numeric IDs or product URLs (mode=byProductIds) |
| `gender` | string | `""` | `""` / `men` / `women` / `boys` / `girls` |
| `sortBy` | string | `""` | Default relevance, newest, popularity, price, discount, or rating |
| `minPrice` | integer | `0` | Minimum selling price in INR; 0 means any |
| `maxPrice` | integer | `0` | Maximum selling price in INR; 0 means any |
| `minDiscount` | integer | `0` | Minimum discount percentage; 0 means any |
| `minRating` | number | `0` | Minimum product rating; products without a rating are excluded when set |
| `containsKeyword` | string | `""` | Keep only product names containing this text |
| `includeDetails` | boolean | `true` | Add sellers, stock, specifications, policies, media, and tags |
| `includeReviews` | boolean | `false` | Emit written-review rows |
| `maxReviewsPerProduct` | integer | `20` | Review rows per product when reviews are enabled |
| `maxItems` | integer | `100` | Total product rows across all inputs (1–10000) |
| `splitLargeSearches` | boolean | `true` | Split large searches by gender, price, and sort for more depth |
| `proxyConfiguration` | object | `{}` | Optional override; leave untouched for automatic connection selection |

## Examples

### Example: keyword search with filters

```json
{
  "mode": "search",
  "searchQueries": ["running shoes", "polo tshirts"],
  "gender": "men",
  "minPrice": 500,
  "maxPrice": 3000,
  "minDiscount": 30,
  "sortBy": "price_asc",
  "maxItems": 200
}
```

### Example: category and brand browse

```json
{
  "mode": "browseByCategory",
  "categories": ["sneakers", "watches"],
  "gender": "women",
  "includeDetails": true,
  "maxItems": 150
}
```

### Example: product URLs and IDs with reviews

```json
{
  "mode": "byUrl",
  "startUrls": [
    "https://www.myntra.com/tshirts/puma/puma-unisex-train-all-day-essentials-slim-fit-drycell-polo-sports-t-shirt/36674095/buy",
    "36674095"
  ],
  "includeReviews": true,
  "maxReviewsPerProduct": 40,
  "maxItems": 20
}
```

### Example: deep export from a listing URL

```json
{
  "mode": "byUrl",
  "startUrls": [
    "https://www.myntra.com/kurtas?f=Gender%3Amen%2Cwomen"
  ],
  "splitLargeSearches": true,
  "sortBy": "popularity",
  "maxItems": 1000
}
```

## Use cases

- **Price and discount monitoring** — track selling price, MRP, coupon price, and discount depth across a catalog
- **Seller intelligence** — see which seller owns the buy box, its location, and its stock position
- **Catalog enrichment** — add fabric, fit, pattern, occasion, and garment measurements to an internal product feed
- **Review analysis** — export ratings, review text, size feedback, photos, and helpfulness votes
- **Assortment research** — compare brands and categories by price band, rating, and available sizes
- **Stock and variant tracking** — monitor per-size availability and total inventory for key styles

## FAQ

### Is any login or API key required?

No. The actor reads publicly available Myntra catalog pages and does not need cookies, accounts, or API keys.

### Do I need to set up a proxy?

No. The actor selects a working connection to Myntra automatically, starting with the cheapest path and escalating only when needed.

### Why do I get fewer products than I requested?

Myntra limits how deep a single listing can be paged. Enable `splitLargeSearches` to partition a large request by gender, price band, and sort order; the actor then removes duplicates. Extremely broad categories can still have practical limits.

### Why are some fields missing on some products?

Fields are omitted when Myntra does not publish them for that product. For example, some listings have no rating yet, some have no video, and some have no coupon.

### What is the difference between `price` and `priceWithCoupon`?

`price` is the current selling price. `priceWithCoupon` is the best price after applying the displayed coupon, and `couponCode` contains that coupon's code.

### What does `detailsFetched` mean?

It is `true` when the product page was opened successfully and the detail fields came from that page. Listing-only modes can still return a product row without it if a detail page is temporarily unavailable.

### Why do review rows not count toward `maxItems`?

`maxItems` controls product rows. Review rows are additional records so that enabling reviews does not reduce the number of products you receive.

### Are the images and videos permanent?

The actor returns the media URLs published by Myntra. They are public links, but Myntra may change or remove media over time.

### Is this affiliated with Myntra?

No. This is an independent third-party actor that reads publicly available catalog pages.
