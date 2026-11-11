# MercadoLibre Mexico Marketplace Price Scraper

MercadoLibre Mexico is the country's largest online marketplace. This actor searches its product listings by keyword and returns price, discount, condition, seller and rating for each result — useful for tracking prices on a specific product line without browsing the site by hand.

## What this actor does

- Searches MercadoLibre Mexico by free-text keyword, or accepts direct MercadoLibre Mexico search URLs to override the structured filters
- Filters results by condition (new/used) and a price range in Mexican pesos
- Captures the discount percentage and original ("before") price when an item is on sale
- Picks up seller name, brand, shipping badge text, and star rating/review count when the listing shows them
- Omits any field the listing didn't display — no empty strings or nulls in the output

## Output per product

- `id` — MercadoLibre item ID (e.g. `MLM1018500855`)
- `title`, `url`
- `countryCode` — always `MX`
- `price` — current price in MXN
- `originalPrice` — pre-discount price, when the item is on sale
- `currency`, `currencySymbol`
- `discountPercentage` — percent off (0–100), when discounted
- `condition` — `new` or `used` (refurbished items are mapped to `used`)
- `thumbnail` — product image URL
- `sellerName`, `brand`
- `shippingLabel` — shipping badge text as shown on the listing (e.g. "Envío gratis")
- `rating`, `reviewsCount`
- `scrapedAt` — UTC scrape timestamp

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `country` | string | `MX` | MercadoLibre country site; only Mexico is currently supported |
| `searchQuery` | string | – | Free-text keyword (required unless `startUrls` is set) |
| `condition` | string | `any` | `any`, `new` or `used` |
| `priceMin` | integer | – | Minimum price in MXN |
| `priceMax` | integer | – | Maximum price in MXN |
| `startUrls` | array | – | Direct MercadoLibre Mexico search URLs, overriding the structured filters above |
| `maxItems` | integer | `50` | Maximum items to return (1–1000) |

### Example: new iPhones under a price cap

```json
{
  "country": "MX",
  "searchQuery": "iphone",
  "condition": "new",
  "priceMax": 20000,
  "maxItems": 30
}
```

### Example: used items in a price band

```json
{
  "country": "MX",
  "searchQuery": "laptop gamer",
  "condition": "used",
  "priceMin": 5000,
  "priceMax": 15000,
  "maxItems": 40
}
```

### Example: a direct search URL

```json
{
  "startUrls": ["https://listado.mercadolibre.com.mx/camara-fotografica"],
  "maxItems": 20
}
```

## Use cases

- **Price monitoring** — run daily to track how a product's listed price moves on MercadoLibre Mexico
- **Competitive research** — compare seller names, shipping offers and ratings across listings for the same keyword
- **Secondary-market analysis** — filter `condition=used` to study the refurbished/used segment
- **Discount tracking** — watch `discountPercentage` to catch newly discounted items
- **Catalog enrichment** — pull thumbnails and titles into a product database by keyword

## FAQ

**Is this affiliated with MercadoLibre?**
No. This is an independent actor that reads publicly available MercadoLibre Mexico listing pages; it is not operated by or affiliated with MercadoLibre.

**Why is only Mexico supported?**
MercadoLibre's other Latin American sites (Argentina, Brazil, Chile, Colombia, Peru and others) currently block automated requests far more aggressively than the Mexico site, so they were left out rather than ship an option that fails every time.

**How do `condition` and the price filters interact?**
Both are applied to the final result set, so a search with `condition=used` and a price range only returns used items whose price falls inside that range.

**Why do some products have no `rating` or `reviewsCount`?**
Not every MercadoLibre listing shows a star rating or review count on the search results page; those fields are only included when the listing actually displays them.

**What happens if MercadoLibre blocks the run?**
On the rare occasion every attempt is blocked, the actor returns a single status record explaining why instead of failing with no output at all. Running again shortly afterward typically succeeds.

**How fresh is the data?**
Each run reads the live MercadoLibre Mexico search results, so prices, discounts and availability reflect what's currently listed.

**Can I target a specific category or saved search instead of a keyword?**
Yes — paste the MercadoLibre Mexico search URL for that category or filter combination into `startUrls`; it overrides `searchQuery`, `condition` and the price fields.
