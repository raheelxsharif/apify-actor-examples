# Global Industrial Product & Pricing Extractor

Pull product and pricing data from Global Industrial (globalindustrial.com), a large US industrial and B2B supplies marketplace covering tools, material handling, HVAC, safety, packaging, electrical, foodservice, and more. Search by keyword, browse a department or category, look up exact item keys, or fetch specific product/category/search URLs.

## What this actor does

- **Four modes:** keyword search, browse a department/category, look up by item key, or fetch by URL
- **Pricing detail:** current price alongside MSR, sale, overstock, and clearance prices, plus quantity-based price breaks
- **Catalog attributes:** brand, SKU, manufacturer part number, availability, lead time, ratings, images, dimensions, and spec attributes
- **Category browsing:** department and category pages, optionally including the subcategory tree
- **Post-fetch filters:** price range, brand, title keyword, minimum rating, and in-stock only
- **Empty fields are omitted, and a failed lookup returns a typed error record instead of breaking the run**

## Output per product (recordType = `product`)

- `itemKey` — Global Industrial's numeric item ID
- `title` — product name
- `productUrl` — product page URL
- `brand`, `sku`, `legacyNumber`, `manufacturerPartNumber`, `unspscCode` — catalog identifiers
- `price` — current listing price
- `priceOriginal`, `priceCatalog`, `priceMsr`, `priceOverstock`, `priceSale`, `priceClearance`, `minSalePrice`, `savings` — other published price points, when applicable
- `priceType` — how the current price is classified
- `autoReOrderDiscount` — auto-reorder discount percentage, when offered
- `priceBreaks` — quantity-tier pricing (`qtyStart`, `qtyEnd`, `price`, `priceOriginal`, `priceType`)
- `availabilityStatus`, `shipTime`, `leadTime`, `inStock`, `freeShipping` — stock and shipping signals
- `avgRating`, `numReviews` — customer rating summary
- `imageUrl`, `imageUrlLarge`, `productImages` — product images at different sizes
- `categoryKey`, `categoryUrl`, `categoryPath`, `breadcrumbs` — the product's category placement
- Flags (present only when true): `isBestSeller`, `isPremium`, `isNewArrival`, `isPrivateLabel`, `isHazardous`, `isNonReturnable`, `isNonCancellable`, `isCallForPrice`, `isMapPrice`, `isCustomizable`, `isFreightFixed`, `isNoAirShip`, `isLtl`, `isReorder`, `isPromotional`
- `dimensions`, `qty`, `attributes` — physical dimensions, pack quantity, and spec attributes
- `description` — product description text
- `itemLaunchDate`, `timeStamp` — catalog dates
- `sourceUrl`, `scrapedAt`

## Output per category (recordType = `category`)

- `categoryKey`, `categoryCode`, `title`, `categoryPath`, `leafUrl`, `imageRef`
- `categoryLevel`, `parentCategoryKey` — the category's position in the catalog tree
- `totalItems`, `sort`, `showMore`, `startIndex`, `lessThan50` — listing metadata for the category
- `categoryUrl`, `heading`, `titleTag`
- `subcategories` — child categories with `categoryKey`, `title`, `url`, `totalItems`, included when `includeSubcategories` is enabled

## Output per failed lookup (recordType = `error`)

- `itemKey` and/or `sourceUrl` — what was being looked up
- `error` — a short reason the lookup failed
- `scrapedAt`

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `search` | `search`, `byCategory`, `byProductIds`, or `byUrls` |
| `searchQuery` | string | `adjustable workbench` | Free-text keyword (mode=`search`) |
| `sortBy` | string | `relevance` | `relevance`, `price_low`, `price_high`, or `best_selling` |
| `categoryPreset` | string | – | A popular department/category shortcut (mode=`byCategory`) |
| `categoryPath` | string | – | Any category path or URL; takes precedence over `categoryPreset` |
| `includeSubcategories` | boolean | `false` | Also emit a category record with the subcategory tree |
| `productIds` | array | – | Item keys, or product URLs ending in `-<key>` (mode=`byProductIds`) |
| `startUrls` | array | – | Product, category, or search URLs to fetch directly (mode=`byUrls`) |
| `minPrice` | number | – | Drop products priced below this amount (USD) |
| `maxPrice` | number | – | Drop products priced above this amount (USD) |
| `minRating` | integer | – | Keep only products rated at or above this value (1–5) |
| `brand` | string | – | Keep only products whose brand contains this text |
| `containsKeyword` | string | – | Keep only products whose title contains this text |
| `inStockOnly` | boolean | `false` | Keep only products marked `InStock` |
| `maxItems` | integer | `50` | Hard cap on product records emitted per run (1–1000) |
| `proxyConfiguration` | object | AUTO, country `US` | The source only serves visitors from the US, so a US-targeted proxy is prefilled |

### Example: keyword search with price and stock filters

```json
{
  "mode": "search",
  "searchQuery": "pallet jack",
  "sortBy": "price_low",
  "minPrice": 100,
  "maxPrice": 500,
  "inStockOnly": true,
  "maxItems": 100
}
```

### Example: browse a department with its subcategories

```json
{
  "mode": "byCategory",
  "categoryPreset": "Power Tools",
  "includeSubcategories": true,
  "maxItems": 200
}
```

### Example: look up specific products by item key

```json
{
  "mode": "byProductIds",
  "productIds": ["33082504", "33201668"]
}
```

### Example: fetch a mix of product, category, and search URLs

```json
{
  "mode": "byUrls",
  "startUrls": [
    { "url": "https://www.globalindustrial.com/p/variable-speed-control-switch-with-6-ft-plug" },
    { "url": "https://www.globalindustrial.com/c/tools/sockets_bits" },
    { "url": "https://www.globalindustrial.com/searchResult?q=shelving" }
  ]
}
```

## Use cases

- **Procurement research** — check current price, availability, and lead time before ordering industrial supplies
- **Price-break analysis** — compare quantity-tier pricing across SKUs for bulk purchasing decisions
- **Category monitoring** — track SKU counts and new-arrival/best-seller flags within a department over time
- **Competitive benchmarking** — compare list, MSR, and clearance pricing across brands within a category
- **Assortment mapping** — build a structured view of Global Industrial's category tree for a specific vertical

## FAQ

**Is this affiliated with Global Industrial?**
No. This is an independent, third-party actor that reads Global Industrial's public product and category pages and is not affiliated with or endorsed by Global Industrial.

**Why does the default proxy target the US?**
Global Industrial's site only serves visitors from US IP addresses; requests from elsewhere are blocked. The actor's prefilled proxy configuration handles this automatically at no extra cost.

**Do I need a login or API key?**
No. All data comes from Global Industrial's publicly viewable catalog pages.

**Why might a category return fewer products than its `totalItems` count?**
Some category pages only render part of their catalog on the page the actor reads; a few landing-page-style categories render no products directly and instead point to subcategories. When this happens, the actor still returns the category and subcategory records it can see.

**Why do some keyword searches return zero products?**
A search that matches a brand name exactly (e.g. `dewalt`) can redirect to a brand landing page whose product grid isn't present in the page the actor reads. Adding a second word to the query (e.g. `dewalt drill`) keeps the search on a normal results page.

**Why does `byProductIds` sometimes return an error record instead of a product?**
Not every item key is present in the lookup path the actor uses. If a key doesn't resolve, the actor emits a typed `error` record instead of silently skipping it. Passing the full product page URL instead of the bare key resolves reliably.

**Are prices current?**
Yes. Each run reads the live catalog pages, so prices, availability, and sale/clearance status reflect what's published at run time.

**Why are some products missing an image?**
If Global Industrial's catalog entry for a product has no image reference, the image fields are left off the record rather than filled with a placeholder.

**What's the difference between `categoryPreset` and `categoryPath`?**
`categoryPreset` is a curated list of common departments and categories you can pick from a dropdown. `categoryPath` accepts any category URL or path directly, including ones not in the preset list, and takes precedence when both are set.
