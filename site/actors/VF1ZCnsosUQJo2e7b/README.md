# Myntra Fashion Catalog Finder by Category & Brand

Myntra's catalog spans thousands of fashion brands across dozens of categories, each with its own pricing, discounts, and size availability. This actor searches that catalog by keyword, browses a category or brand, or fetches specific product pages directly, returning pricing, ratings, and variant detail for every match. No login required.

## What this actor does

- **Four modes**: keyword search, browse by category, browse by brand, or fetch specific product URLs
- **Covers 48 curated categories** — from T-shirts and jeans to sarees, sneakers, and jewellery
- **Returns pricing and discount detail** — MRP, selling price, discount amount and percentage
- **Filters before you see results** — gender, sort order, price range, minimum rating, and a keyword match on the product name
- **Optional detail enrichment** — fetch each product's own page for description, material & care, and manufacturer info
- **Empty fields are omitted** from each record instead of being returned as null

## Output per product

- `productId` — Myntra's product/style ID
- `productName`, `brand`, `additionalInfo` — naming
- `mrp`, `price` — list price and selling price, in INR
- `discountAmount`, `discountPercent`, `discountDisplayLabel` — discount detail (e.g. `(51% OFF)`)
- `rating`, `ratingCount` — average rating and number of ratings
- `primaryColour`, `gender`, `season`, `year` — product attributes
- `category`, `subCategory`, `masterCategory`, `categoryPath` — Myntra's category hierarchy
- `sizes[]`, `hasMultipleSizes` — available size labels
- `imageUrl`, `imageUrls[]` — product images
- `colourVariantImages[]`, `relatedColoursCount` — other color variants of the same product
- `isFastFashion` — Myntra's fast-fashion flag
- `productUrl`, `sourceUrl` — canonical product URL
- `recordType` — always `"product"`
- `scrapedAt` — timestamp of when the record was scraped

When `includePdpDetails` is enabled (or in `byUrl` mode, which always fetches the product page), records also include:

- `description`, `materialCare`, `sizeFit` — detail-page copy
- `manufacturer`, `countryOfOrigin`, `baseColour`
- `reviewsCount`, `pdpViews`
- `availableSizes[]` — sizes currently in stock
- `colours[]` — other colors, each with `styleId`, `label`, `imageUrl`, `productUrl`
- `offers[]` — active promotional offers on the product
- `descriptor_<section>` — extra labeled detail sections Myntra publishes on the product page (e.g. a fit or wash-care section); field name depends on the section title. Only present in `byUrl` mode, not when `includePdpDetails` enriches a listing

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string (enum, required) | `search` | `search`, `browseByCategory`, `browseByBrand`, or `byUrl` |
| `searchQuery` | string | `tshirts` | Free-text keyword (mode `search`) |
| `category` | string (enum) | `tshirts` | One of 48 curated category slugs (mode `browseByCategory`) |
| `brandName` | string | – | Brand name as it appears on Myntra, e.g. `adidas`, `levis` (mode `browseByBrand`) |
| `productUrls` | array | `[]` | Full Myntra product URLs to fetch directly (mode `byUrl`) |
| `gender` | string (enum) | `` (any) | `men`, `women`, `boys`, or `girls` |
| `sortBy` | string (enum) | `` (default) | `recommended`, `popularity`, `new`, `price_asc`, `price_desc`, `discount`, or `customer_rating` |
| `minPrice` | integer | – | Drop products below this selling price (INR) |
| `maxPrice` | integer | – | Drop products above this selling price (INR) |
| `minRating` | number | – | Drop products rated below this value (0–5) |
| `containsKeyword` | string | – | Only keep products whose name contains this keyword |
| `includePdpDetails` | boolean | `false` | Also fetch each product's detail page for description, material, and manufacturer |
| `maxItems` | integer | `50` | Maximum number of records to return (1–1000) |
| `proxyConfiguration` | object | off | Optional Apify proxy; engaged automatically only if requests get blocked |

### Example: filtered keyword search

```json
{
  "mode": "search",
  "searchQuery": "printed tshirts",
  "gender": "men",
  "minPrice": 300,
  "maxPrice": 1000,
  "minRating": 4,
  "maxItems": 20
}
```

### Example: browse a category sorted by popularity

```json
{
  "mode": "browseByCategory",
  "category": "sneakers",
  "sortBy": "popularity",
  "maxItems": 30
}
```

### Example: a brand's catalog with full detail

```json
{
  "mode": "browseByBrand",
  "brandName": "adidas",
  "includePdpDetails": true,
  "maxItems": 10
}
```

### Example: fetch specific product pages

```json
{
  "mode": "byUrl",
  "productUrls": [
    "https://www.myntra.com/tshirts/keesor/keesor-girls-pack-of-3-multi-cotton-tshirts/36385830/buy"
  ]
}
```

## Use cases

- **Price tracking** — monitor MRP versus selling price and discount percentage across a brand or category
- **Competitive monitoring** — watch new arrivals and price moves for competing fashion brands
- **Assortment research** — export a category's full catalog with sizes, colors, and ratings
- **Merchandising and recommendation** — feed product names, categories, and images into your own storefront
- **Market studies** — analyze Indian fashion e-commerce pricing and assortment patterns

## FAQ

**Is this affiliated with Myntra?**
No. This is an independent, third-party actor that reads Myntra's publicly visible product pages; it isn't operated by or affiliated with Myntra.

**Do I need an account or proxy?**
No. The actor runs without cookies or an account; a proxy is only engaged automatically if Myntra blocks a request.

**Why does a listing mode return a capped number of products rather than paginating further?**
Myntra's public listing pages render a single page of results per request; deeper pagination isn't publicly available. For a larger export, split your run by brand, category, or gender instead.

**What happens if I search using non-Latin characters (e.g. Chinese or Japanese text)?**
The actor falls back to Myntra's general search page for such queries. If Myntra has no matching inventory for that query, the run returns zero records with a status message rather than erroring out.

**What happens if I enter a brand that isn't on Myntra?**
Myntra can redirect an unknown brand name to an unrelated listing page. The actor detects this redirect and returns zero records with a "brand not found" message instead of emitting mismatched products.

**What currency are prices in?**
Indian Rupees (INR), as shown on myntra.com.

**Why are fields like `rating` or `offers` sometimes missing?**
Not every product has a rating or an active offer. Fields with no value on the source page are omitted from the record rather than returned empty.

**How fresh is the data?**
Each run reads Myntra's live pages, so prices, discounts, and availability reflect what's live at run time.
