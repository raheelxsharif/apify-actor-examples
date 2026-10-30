# Lowe's Price, Stock & Specification Scraper

Lowe's sells hundreds of thousands of home-improvement products, each with its own price, stock status, and spec sheet. This actor pulls that data by keyword search, category browse, item ID, or a direct product URL, so you can track pricing and availability without visiting the site page by page.

## What this actor does

- Searches Lowe's products by keyword, with a choice of sort order.
- Browses a product category (Tools, Electrical, Plumbing, Appliances, and more).
- Fetches full product details for one or more Lowe's item numbers.
- Scrapes any Lowe's product or search URL you provide directly.
- Filters results by stock status, price range, and minimum rating.
- Leaves out fields a product listing doesn't have (no discount, no store availability message) rather than sending them empty.

## Output per product

- `title`, `url`, `itemId`
- `brand`, `model`
- `price`, `originalPrice` — current price and the pre-discount price, when the item is on sale
- `discountPercent` — calculated when both prices are present
- `inStock` — whether the item is currently in stock
- `storeAvailability` — store-level stock message, when shown
- `rating`, `reviewCount`
- `imageUrl`
- `description`, `specifications` — full description and a list of `{name, value}` spec pairs (from item-ID and URL lookups)
- `category`
- `scrapedAt`

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `search` | `search`, `byCategory`, `productDetails`, or `byUrl` |
| `searchQuery` | string | `power drill` | Keyword to search for; used by `search` |
| `category` | string | `tools` | Category to browse; used by `byCategory` |
| `itemIds` | array | `[]` | Lowe's item numbers, e.g. `5013654453`; used by `productDetails` |
| `startUrls` | array | `[]` | Lowe's product or search page URLs; used by `byUrl` |
| `sortBy` | string | `bestMatch` | `bestMatch`, `topRated`, `priceAsc`, `priceDesc`, or `newArrivals` |
| `inStockOnly` | boolean | `false` | Only return products currently in stock |
| `minPrice` | integer | – | Skip products priced below this (USD) |
| `maxPrice` | integer | – | Skip products priced above this (USD) |
| `minRating` | number | – | Only return products rated at or above this (1.0-5.0) |
| `proxyConfiguration` | object | `{"useApifyProxy": true}` | Apify Proxy configuration for the run |
| `maxItems` | integer | `20` | Maximum number of products to return (1-500) |

### Example: keyword search

```json
{
  "mode": "search",
  "searchQuery": "power drill",
  "maxItems": 20
}
```

### Example: browse a category, in-stock only, sorted by price

```json
{
  "mode": "byCategory",
  "category": "appliances",
  "inStockOnly": true,
  "sortBy": "priceAsc",
  "maxItems": 50
}
```

### Example: look up specific items

```json
{
  "mode": "productDetails",
  "itemIds": ["5013654453"]
}
```

## Use cases

- Monitor price and discount changes on specific products or whole categories.
- Build a stock-availability tracker for items you need to restock.
- Compare specifications and ratings across competing products before buying.
- Feed Lowe's pricing data into a broader retail price-comparison tool.

## FAQ

**Is this affiliated with Lowe's?**
No. This actor reads Lowe's own public product pages and has no relationship with Lowe's.

**Do I need a Lowe's account?**
No. Every field comes from pages that are publicly visible without logging in.

**Why does this actor need a proxy?**
`proxyConfiguration` is enabled by default because direct requests to Lowe's are unreliable without it; leave it on unless you have your own proxy setup to supply instead.

**Why do some products have no `specifications` or `description`?**
Those fields are only populated when a product is fetched by item ID or URL (`productDetails`/`byUrl`); search and category results carry a lighter set of fields.

**How many products can I get per run?**
Up to `maxItems`, capped at 500 per run.

**How current is the data?**
Each run reads Lowe's live, so price, discount, and stock status reflect what the site shows at run time.
