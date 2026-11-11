# Target Price, Stock & Deals Finder

Pull live product data straight from Target.com — keyword search, a whole department, exact item numbers, a pasted product link, or the current deals rail. Every record carries pricing, sale/clearance flags, online availability, ratings, UPC/DPCI identifiers, category breadcrumbs and product specifications.

## What this actor does

- **Five modes:** `search` by keyword, `byCategory` to browse a department, `byTcin` to fetch exact items, `byUrl` to resolve a pasted Target link, and `deals` for Target's current deals
- **Rich pricing:** current price, regular price, currency, savings, discount percent and price type (`reg`, `sale`, `clearance`; marked-down product pages without a Target label are reported as `sale`)
- **Availability:** online stock status plus a shipping flag when Target marks the item as shippable; listings with no stock signal are reported as `UNKNOWN`
- **Product identifiers:** TCIN, UPC, DPCI, category ID, category name and breadcrumbs
- **Review signal:** average rating, rating count and written-review count
- **Product content:** description, specifications, feature bullets, primary image and direct product URL
- **Server-side sort:** relevance, featured, price low/high, highest rated, best sellers and newest
- **Filters:** price range, minimum rating, on-sale-only and in-stock-only, applied before records count toward `maxItems`
- **Pagination:** follows Target's result pages up to 50 pages per query (about 1,200 products), deduplicating by TCIN
- **Empty fields are omitted** from each record, so a product without a brand or without review data simply won't have those keys

## Output per product

- `tcin` — Target item number, the unique product identifier
- `title` — product name
- `brand` — brand name
- `description` — product description text (with full-detail enrichment)
- `price` — current selling price (USD)
- `originalPrice` — regular/list price (USD)
- `priceCurrency` — currency code, normally `USD`
- `priceType` — `reg`, `sale`, `clearance` or another Target price label; `sale` is inferred when a product page is marked down but carries no label
- `isOnSale` — `true` when the item is on sale or clearance
- `saleSavings` — dollar amount saved versus the regular price
- `discountPercent` — percent saved versus the regular price
- `rating` — average customer rating (0–5)
- `ratingCount` — total number of customer ratings
- `reviewCount` — number of written reviews
- `dpci` — Target department/class/item number (full-detail enrichment)
- `upc` — product barcode (full-detail enrichment)
- `categoryId` / `categoryName` — Target category identifiers
- `breadcrumbs[]` — category path with names, IDs and URLs
- `specifications` — product specification key/value pairs (full-detail enrichment)
- `featureBullets[]` — specification and feature bullets
- `primaryImageUrl` — main product image URL
- `availability` — `IN_STOCK`, `LIMITED_STOCK`, `OUT_OF_STOCK`, or `UNKNOWN` (search, department and deals listings do not expose a stock signal; `includeDetails` resolves it from the product page)
- `fulfillment` — `{ shipping, pickup, sameDay }` booleans when Target exposes them
- `position` — 1-based result position in this run
- `productUrl` / `sourceUrl` — direct link to the product's Target.com page
- `recordType` — always `"product"`
- `siteName` — always `"Target"`
- `scrapedAt` — ISO 8601 timestamp of when the record was collected

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `search` | `search`, `byCategory`, `byTcin`, `byUrl`, or `deals` |
| `query` | string | `iPhone case` | Search keyword (mode=`search`) |
| `tcins` | array | – | Target item numbers to fetch (mode=`byTcin`) |
| `categoryId` | string | `electronics` | Department to browse (mode=`byCategory`) |
| `url` | string | – | A Target product, category, brand or search URL (mode=`byUrl`) |
| `sortBy` | string | `relevance` | `relevance`, `featured`, `priceLow`, `priceHigh`, `ratingHigh`, `bestSeller`, `newest` |
| `includeDetails` | boolean | `true` | Fetch each product page for description, UPC/DPCI, breadcrumbs, specifications and online stock status |
| `minPrice` | number | – | Drop products priced below this USD amount |
| `maxPrice` | number | – | Drop products priced above this USD amount |
| `minRating` | number | – | Drop products with average rating below this value (0–5) |
| `onSaleOnly` | boolean | `false` | Only return products marked on sale or clearance |
| `inStockOnly` | boolean | `true` | Exclude products Target reports as out of stock; listings with no stock signal are kept and marked `UNKNOWN` |
| `maxItems` | integer | `24` | Hard cap on returned products (1–1000) |
| `maxPages` | integer | `10` | Maximum result pages per query (1–50) |

### Example: keyword search with a price ceiling

```json
{
    "mode": "search",
    "query": "bluetooth headphones",
    "maxPrice": 100,
    "maxItems": 48
}
```

### Example: look up exact products by item number

```json
{
    "mode": "byTcin",
    "tcins": ["94715153", "12345678"]
}
```

### Example: browse a department for on-sale items

```json
{
    "mode": "byCategory",
    "categoryId": "home",
    "onSaleOnly": true,
    "maxItems": 100
}
```

### Example: resolve a pasted product link

```json
{
    "mode": "byUrl",
    "url": "https://www.target.com/p/apple-airpods-pro/-/A-85978612"
}
```

### Example: trending deals

```json
{
    "mode": "deals",
    "maxItems": 24,
    "sortBy": "bestSeller"
}
```

## Use cases

- **Price monitoring** — track `price`, `originalPrice`, `priceType` and `saleSavings` for a watchlist of TCINs over time
- **Assortment research** — browse a department with `byCategory` to see what's listed right now (add `includeDetails` for per-item stock status)
- **Deal discovery** — run the `deals` mode on a schedule to catch newly featured promotions
- **Review benchmarking** — compare `rating`, `ratingCount` and `reviewCount` across competing products
- **Catalog enrichment** — resolve saved Target URLs into structured records with UPC, DPCI, specifications and category breadcrumbs
- **Identifier matching** — join Target products to other catalogs by TCIN, UPC or DPCI

## FAQ

**Where does this data come from?**
Target's own publicly crawlable product pages — the server-rendered version of Target.com that search engines index. Search, department and deals pages expose titles, prices, ratings and identifiers; product pages add stock status, UPC/DPCI, breadcrumbs and specifications. The actor needs no Target account, cookie or API key.

**What is a TCIN?**
TCIN stands for Target Corporation Item Number — Target's internal product identifier, visible in product URLs after `/A-`.

**Do I need a Target account or a proxy?**
No. All modes read publicly listed product data without a login, cookie or API key. The actor picks its network route automatically.

**What's the difference between the five modes?**
`search` and `byCategory` return lists of products — one from a keyword query, the other from a department keyword. `byTcin` and `byUrl` return exact products, addressed by item number or a pasted link. `deals` returns Target's current deals.

**Why is a field like `brand` or `rating` sometimes missing?**
Target doesn't populate every field for every listing — some products have no brand attribution, and newly listed items may not yet have reviews. Missing fields are omitted rather than returned empty.

**What does `includeDetails` do?**
It fetches each product page to add the description, UPC/DPCI, category breadcrumbs, specifications and the online stock status Target publishes. It is on by default; turn it off for the fastest, lightest run. The daily test uses the light path.

**Why is `availability` sometimes `UNKNOWN`?**
Target's search, department and deals listings (the pages this actor reads without extra requests) do not publish a per-item stock status. Rather than guessing, the actor marks those records `UNKNOWN`; enable `includeDetails` or look up the item with `byTcin`/`byUrl` to get the status from its product page. The `inStockOnly` filter still removes every item Target explicitly reports as out of stock.

**Can I combine the price, rating, and stock filters?**
Yes. `minPrice`, `maxPrice`, `minRating`, `onSaleOnly` and `inStockOnly` all apply together, and only products passing every filter count toward `maxItems`.

**How many products can one run return?**
Up to 1,000, set by `maxItems`, and at most 50 result pages per query (`maxPages`). Target caps a search at about 1,200 products; for broader coverage, run again with a different `query`, `categoryId` or `sortBy`.

**Why do some runs return fewer products than `maxItems`?**
Target may have fewer matches, the filters may reject some, or the source may reach its 50-page ceiling. The actor never fabricates rows to fill the quota.

**Does the actor return store-level pickup stock?**
No. Target's per-store stock API is protected by an anti-bot challenge and is not reachable from this actor's network. `availability` reflects Target's public online stock status (and is `UNKNOWN` when the source page carries no signal); store-level pickup quantities are not included.

**How reliable is the data route?**
The actor reads Target's public server-rendered pages and tries the cheapest network route first, escalating automatically if a response lacks the expected product data. This path is public and cookieless, but it is a search-engine view of the catalogue: it can lag the interactive site and Target may change its markup. The actor never invents records to fill a quota — a run reports fewer items rather than fabricating them.

**Are product variations included?**
Target's public product page exposes the selected variation's price and identifiers but not the full variation list. The actor returns the item you asked for; it does not enumerate every color or size.

**Is this affiliated with Target?**
No. This is an independent, third-party actor and is not operated by or affiliated with Target Corporation.
