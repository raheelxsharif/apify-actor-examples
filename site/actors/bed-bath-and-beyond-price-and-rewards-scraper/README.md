# Bed Bath & Beyond Price & Rewards Scraper

Search the current [bedbathandbeyond.com](https://www.bedbathandbeyond.com) catalog by keyword or browse a home-goods department — bedding, bath, kitchen, furniture, decor and more. Each product comes back with live price and sale price, brand, customer rating, two-day-shipping eligibility and Welcome Rewards cash-back figures.

## What this actor does

- Searches the catalog by free-text keyword, or browses one of 16 real home-goods departments
- Returns live USD pricing, including sale price, original MSRP and the discount percentage
- Reports customer rating, review count, two-day-shipping eligibility and Welcome Rewards cash-back data
- Filters by price range, minimum rating, on-sale-only, two-day-shipping-only, and a brand/title text match
- Sorts by relevance, on-sale, best selling, price, rating or new arrivals
- Only fields with a real value are included — nothing is emitted as a placeholder

## Output per product

- `title` — product name
- `brand` — manufacturer/brand name, matched from the catalog's own brand list for that query
- `productId`, `sku` — internal catalog identifiers
- `price`, `priceMax`, `currency` — current price; `priceMax` is set when the product has size/color options priced differently, with `price` as the low end
- `originalPrice` — list/MSRP price before any discount
- `discountPercent` — percent off `originalPrice`
- `onSale` — whether the product is currently marked on sale
- `saleEndDate` — when the current sale price expires (only present when `onSale` is true)
- `urgencyMessaging`, `valueMessaging` — merchandising badges such as "Best Seller"
- `rating`, `reviewCount` — average customer rating (0–5) and number of reviews
- `optionCount` — number of color/size variants available
- `twoDayShipping` — whether the product qualifies for two-day delivery
- `rewardsAmount`, `rewardsPoints`, `rewardsPercentage` — Welcome Rewards cash-back amount, points earned, and the rate applied
- `imageUrl`, `fullImageUrl` — standard and full-resolution product photo
- `productUrl`, `sourceUrl` — canonical product page URL
- `isSponsored` — whether the listing is a sponsored placement
- `category` — the department you browsed (present only in `byCategory` mode)
- `searchKeywords` — the keyword used (present only in `search` mode)
- `recordType: "product"`, `scrapedAt`

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `search` | `search` (keyword) or `byCategory` (browse a department) |
| `keywords` | string | – | Free-text query, e.g. `towel`, `comforter set`, `kitchen island` (mode=search) |
| `category` | string | `Bedding` | One of 16 departments: Bedding, Bath, Kitchen and Dining, Furniture, Home Decor, Storage and Organization, Home Improvement, Appliances, Baby and Kids Home Goods, Outdoor, Rugs, Housewares, Holiday, Art, Gifts, Tools (mode=byCategory) |
| `sortBy` | string | `relevance` | `relevance` / `onSale` / `bestSelling` / `priceLowHigh` / `priceHighLow` / `rating` / `newArrivals` |
| `minPrice` | integer | – | Drop products priced below this amount (USD) |
| `maxPrice` | integer | – | Drop products priced above this amount (USD) |
| `minRating` | integer | – | Drop products rated below this (0–5) |
| `onSaleOnly` | boolean | `false` | Only emit products currently on sale |
| `fastShippingOnly` | boolean | `false` | Only emit two-day-shipping-eligible products |
| `brandContains` | string | – | Keep only products whose brand or title contains this text |
| `maxItems` | integer | `30` | Hard cap on emitted records (1–1000) |

### Example: keyword search

```json
{
  "mode": "search",
  "keywords": "comforter set",
  "maxItems": 30
}
```

### Example: browse a department, on-sale only, sorted by rating

```json
{
  "mode": "byCategory",
  "category": "Kitchen and Dining",
  "sortBy": "rating",
  "onSaleOnly": true,
  "maxItems": 40
}
```

### Example: keyword search with price and shipping filters

```json
{
  "mode": "search",
  "keywords": "area rug",
  "minPrice": 50,
  "maxPrice": 300,
  "fastShippingOnly": true,
  "maxItems": 25
}
```

## Use cases

- **Price monitoring** — track sale prices and discount percentages for products or a whole department over time
- **Assortment research** — see what's trending, best-selling, or on sale within a department
- **Competitive analysis** — compare catalog pricing against other home-goods retailers
- **Content and affiliate sites** — source product data, images, and ratings for buying guides
- **Deal aggregation** — surface highly rated, on-sale products across departments

## FAQ

**Is bedbathandbeyond.com still a real, active store?**
Yes. The Bed Bath & Beyond brand relaunched as an online-only storefront after its original operator's bankruptcy; this actor reads that current, live catalog.

**Do I need a proxy or login?**
No. The catalog's search results are publicly accessible with no account or cookies required.

**Why does `byCategory` mode browse by searching the department name instead of a dedicated category page?**
Direct department-browse pages aren't reliably reachable for automated requests, so this actor reaches each department through the catalog's own search for that department name instead. The 16 departments listed above are the real, verified top-level departments the catalog itself returns.

**Why is `category` only present in `byCategory` mode?**
Individual search-result listings don't carry their own category field — `category` simply records which department you asked for, so it's only meaningful (and only populated) when you're in `byCategory` mode.

**Why is `brand` sometimes missing?**
`brand` is detected by matching the product title against the catalog's own brand list for that query. Generic or unbranded listings won't match any brand and are returned without this field.

**Does `sortBy` reorder the entire catalog or just what was fetched?**
For `priceLowHigh`, `priceHighLow`, `rating` and `onSale`, the actor reorders the window of results it fetched for your query — not the full multi-thousand-item catalog. For the true lowest- or highest-priced item across a whole department, pair a price sort with a larger `maxItems`. `bestSelling` and `newArrivals` have no equivalent signal in the fetched data, so those fall back to the catalog's own relevance order.

**What currency are prices in?**
USD only — this actor targets the US storefront.

**Are the image URLs stable, or do they expire?**
They're plain CDN URLs with no signing or expiration, so they keep working after the run finishes.

**Is this affiliated with Bed Bath & Beyond?**
No. This is an independent, third-party actor that reads the publicly visible catalog; it is not affiliated with or endorsed by Bed Bath & Beyond or its operating company.
