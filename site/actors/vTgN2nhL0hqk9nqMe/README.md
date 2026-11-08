# Best Buy Price Drop & Stock Tracker

Pull structured product data straight off Best Buy.com by typing a search keyword, pasting product page URLs, or both. Each result is a flat record carrying current price, regular price, sale savings, rating, review count, brand, model and stock status — handy for watching a shopping list for a price drop or a restock.

## What this actor does

- Searches Best Buy by free-text keyword (product name, brand or category) and collects matching product pages
- Accepts specific Best Buy product URLs directly, in addition to or instead of a keyword
- Computes savings amount, savings percent and an `onSale` flag whenever a regular price and a lower current price both exist
- Can skip items currently marked out-of-stock or sold out
- Empty fields are omitted from records — a field only appears when Best Buy actually published that data

## Output per product

- `sku` — Best Buy's product SKU, parsed from the product URL
- `url` — link to the product detail page that was scraped
- `name` — product title
- `description` — marketing description text
- `brand` — manufacturer name
- `model` — model number / MPN
- `gtin13` — GTIN-13 / barcode, when published
- `image` / `imageUrl` — primary product image URL (same value under both keys)
- `currentPrice` — price shown to the shopper at scrape time
- `regularPrice` — non-sale list price
- `salePrice` — promotional price, when the item is on sale
- `savings` — `regularPrice` minus the lower of `salePrice`/`currentPrice`
- `savingsPercent` — savings expressed as a percentage of `regularPrice`
- `onSale` — `true` when a sale price below the regular price was detected
- `currency` — ISO currency code of the prices
- `currencySymbol` — display symbol matching `currency` (e.g. `$`)
- `availability` — stock status as published by Best Buy, e.g. `InStock`, `OutOfStock`, `SoldOut`, `PreOrder`, `BackOrder` or `LimitedAvailability`
- `seller` — name of the seller fulfilling the listing (usually Best Buy, sometimes a marketplace seller)
- `rating` — average customer rating
- `reviewCount` — number of customer reviews behind the rating
- `scrapedAt` — ISO-8601 UTC timestamp of when the record was collected

## Output: no-results notice (`type: "bestbuy_error"`)

Pushed instead of product records when nothing could be parsed for the given keyword/URLs (typically a persistent bot-challenge).

- `reason` — short code, e.g. `no_results`
- `message` — human-readable explanation
- `keyword`, `productUrls` — the input that produced no results
- `scrapedAt`

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `keyword` | string | – | Free-text search — product name, brand or category. Use `keyword`, `productUrls`, or both. |
| `productUrls` | array | `[]` | Specific Best Buy product page URLs to scrape directly |
| `maxResults` | integer | `24` | Maximum number of product records to return (1–240), combining keyword search and URL results |
| `skipOutOfStock` | boolean | `false` | When `true`, drop products currently marked out-of-stock or sold out |

### Example: track a watch-list of exact products

```json
{
  "productUrls": [
    "https://www.bestbuy.com/site/sony-wh-1000xm5-wireless-noise-canceling-over-the-ear-headphones-black/6505729.p?skuId=6505729"
  ],
  "skipOutOfStock": false
}
```

### Example: scan a category for in-stock deals

```json
{
  "keyword": "sony headphones",
  "maxResults": 48,
  "skipOutOfStock": true
}
```

### Example: combine search and specific SKUs in one run

```json
{
  "keyword": "robot vacuum",
  "productUrls": [
    "https://www.bestbuy.com/site/irobot-roomba-694-wi-fi-connected-robot-vacuum/6485800.p?skuId=6485800"
  ],
  "maxResults": 60
}
```

## Use cases

- **Price-drop alerts** — re-run against the same `productUrls` on a schedule and diff `currentPrice` over time
- **Restock monitoring** — pair `skipOutOfStock: false` with the `availability` field to catch when a sold-out item comes back
- **Category research** — search a keyword like "robot vacuum" to see every matching listing's price and rating side by side
- **Competitive pricing** — compare Best Buy's `currentPrice` and `savingsPercent` against the same SKU on other retailers
- **Review trend tracking** — snapshot `rating` and `reviewCount` over successive runs to see how reception changes

## FAQ

**Do I need a keyword and URLs, or just one?** Either works on its own, and you can combine them — `productUrls` results are included alongside whatever the keyword search finds, up to `maxResults`.

**Why is a field like `salePrice` or `gtin13` missing from a record?** Best Buy doesn't publish every field on every product page. If a field wasn't present on the page, it's left out of the record instead of being returned empty or null.

**What does `onSale` mean exactly?** It's set to `true` only when the actor finds a sale/current price that is actually lower than the regular price on the same page — not just whenever a `salePrice` field exists.

**Can I scrape an entire category page?** `productUrls` expects individual product page URLs, not category/collection pages. To cover a whole category, run a `keyword` search instead — Best Buy's own search matches category and collection terms too.

**How is `currentPrice` different from `regularPrice` and `salePrice`?** `currentPrice` is whatever price is shown to a shopper right now. `regularPrice` is the non-discounted list price, and `salePrice` only appears when the item is actively on promotion.

**Is this the official Best Buy API?** No. This actor reads the publicly visible Best Buy product pages; it is not affiliated with or endorsed by Best Buy.

**Why did some SKUs come back with no record at all?** A product page that failed to load or didn't contain enough identifying data (no name and no SKU) is skipped rather than pushed as an empty record.
