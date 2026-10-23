# Hepsiburada Product Search & Category Price Scraper

Pull live product data from Hepsiburada, Turkey's largest multi-category marketplace, four different ways: keyword search, a category page, a list of product URLs, or a keyword's top sellers — each returning structured price, rating, and seller records.

## What this actor does

- Searches Hepsiburada by keyword and returns matching products with current price, rating, and seller.
- Browses any category page URL and returns its product listing, sorted the same way the site's own sort options work.
- Fetches full detail (including the product description) for a specific list of product page URLs.
- Surfaces the top-selling products for a keyword using Hepsiburada's own popularity ranking.
- Lets every mode be filtered to a price range in Turkish Lira and capped to a maximum item count; fields Hepsiburada doesn't publish for a product are left out of its record.

## Output per product

| Field | Type | Description |
|---|---|---|
| `productId` | String | Hepsiburada product ID (SKU) |
| `title` | String | Product title |
| `brand` | String | Brand name |
| `description` | String | Product description (`productDetails` mode only) |
| `category` | String | Product category |
| `price` | Number | Current price (TRY) |
| `originalPrice` | Number | Original price before discount (TRY) |
| `discountPercent` | Integer | Discount percentage (0–100) |
| `rating` | Number | Average rating (0–5 scale) |
| `reviewCount` | Integer | Number of customer reviews |
| `seller` | String | Seller/merchant name |
| `freeShipping` | Boolean | Whether free shipping is available |
| `isMerchant` | Boolean | `true` if sold by a marketplace seller, `false` if sold directly by Hepsiburada |
| `imageUrl` | String | Product image URL |
| `productUrl` | String | Full Hepsiburada product page URL |
| `recordType` | String | Always `product` |
| `scrapedAt` | String | ISO 8601 timestamp of when the record was scraped |

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `search` | `search`, `productDetails`, `categoryBrowse`, or `topSellers` |
| `keyword` | string | – | Search keyword (required for `search` and `topSellers`) |
| `productUrls` | array | `[]` | Hepsiburada product page URLs (required for `productDetails`) |
| `categoryUrl` | string | – | Hepsiburada category page URL (required for `categoryBrowse`) |
| `sortBy` | string | `MOST_POPULAR` | `MOST_POPULAR`, `LOWEST_PRICE`, `HIGHEST_PRICE`, `MOST_RECENT`, or `BEST_RATED` |
| `minPrice` | integer | – | Minimum price in Turkish Lira |
| `maxPrice` | integer | – | Maximum price in Turkish Lira |
| `maxItems` | integer | `20` | Maximum number of products to return (1–500) |
| `proxyConfiguration` | object | – | Apify proxy settings; recommended for consistent access |

### Example: keyword search sorted by price

```json
{
  "mode": "search",
  "keyword": "laptop",
  "sortBy": "LOWEST_PRICE",
  "maxItems": 20
}
```

### Example: detail lookup for specific products

```json
{
  "mode": "productDetails",
  "productUrls": [
    "https://www.hepsiburada.com/apple-macbook-pro-14-pm-HBCV00000GVYH5"
  ]
}
```

### Example: browse a category page within a price range

```json
{
  "mode": "categoryBrowse",
  "categoryUrl": "https://www.hepsiburada.com/laptop-x-c-1",
  "sortBy": "LOWEST_PRICE",
  "minPrice": 20000,
  "maxPrice": 60000,
  "maxItems": 50
}
```

## Use cases

- **Price monitoring** — track a product line's price and discount percentage over time
- **Market research** — compare brands, ratings, and review counts within a category
- **Competitor tracking** — watch which products a merchant lists and how they're priced
- **Assortment analysis** — pull a whole category to see Hepsiburada's live product mix
- **Trend spotting** — use `topSellers` to see which products currently rank best for a keyword

## FAQ

**What's the difference between `search` and `topSellers`?**
`search` returns matching products in whatever order `sortBy` specifies; `topSellers` always ranks by Hepsiburada's own popularity signal for the keyword, regardless of `sortBy`.

**Why does `description` only appear sometimes?**
It's only populated in `productDetails` mode, where the actor fetches each product's own page; search and category results don't include the full description.

**What is `isMerchant`?**
Hepsiburada sells both as a direct retailer and as a marketplace for third-party sellers. `isMerchant` is `true` when the listing is fulfilled by a marketplace seller and `false` when Hepsiburada is the seller of record.

**Can I filter by price?**
Yes — `minPrice` and `maxPrice` apply to any mode and are both in Turkish Lira (TRY).

**How do I find a category URL for `categoryBrowse`?**
Copy the URL of any Hepsiburada category listing page, for example `https://www.hepsiburada.com/laptop-x-c-1` or `https://www.hepsiburada.com/cep-telefonu-x-c-32`.

**Do I need to log in or provide cookies?**
No — all four modes read Hepsiburada's public pages.

**Why would I need the proxy setting?**
Hepsiburada's anti-bot protection can block repeated requests from a single IP; enabling the proxy input improves reliability if you see failed or empty results.

**Is this affiliated with Hepsiburada?**
No. This is an independent, third-party actor that reads Hepsiburada's public pages and is not operated by or affiliated with Hepsiburada.
