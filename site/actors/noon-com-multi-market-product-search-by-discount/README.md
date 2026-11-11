# Noon.com Multi-Market Product Search by Discount

Search **noon.com** across the UAE, Saudi Arabia, Egypt, Bahrain, Qatar, Kuwait, and Oman, then export structured product rows with local-currency prices, list prices, discounts, ratings, sellers, stock, images, specifications, and recent review snippets. Four modes cover keyword search, category browsing, direct SKU or product-URL lookups, and pasted noon URLs — in English or Arabic, with no login, cookie, API key, or proxy setup.

## What this actor does

- **Four modes:** keyword `search`, `byCategory`, `byProduct` (SKU or product URL), and `byUrl` (search, category, or product URL)
- **Seven GCC storefronts:** UAE (AED), Saudi Arabia (SAR), Egypt (EGP), Bahrain (BHD), Qatar (QAR), Kuwait (KWD), Oman (OMR)
- **Discount search:** filter by minimum or maximum discount percentage, not just noon's default ranking
- **Product filters:** price range, rating, brand, seller, and deal-only
- **Listing and detail data:** seller, stock, images, specifications, offers, warranty, delivery, and up to five latest reviews
- **Empty fields are omitted** — every record contains only fields that have real values

## Output per product

- `sku` — noon product identifier (e.g. `N70367827V`)
- `skuConfig` — noon SKU configuration/offer identifier when present
- `name` — product title
- `brand`, `brandCode` — brand name and noon brand code
- `url` — direct noon product page URL
- `price` — current selling price in the market's currency
- `originalPrice` — pre-discount list price, when discounted
- `discountPercent` — calculated discount percentage, when discounted
- `discountAmount` — calculated saving, when discounted
- `currency` — `AED`, `SAR`, `EGP`, `BHD`, `QAR`, `KWD`, or `OMR`
- `rating` — average product rating (0–5)
- `reviewCount` — total ratings/reviews shown by noon
- `recentReviews` — up to five latest review snippets (with `includeReviews`)
- `seller`, `sellerCode` — buy-box seller name and code
- `sellerRating`, `sellerReviewCount`, `sellerPositivePercent` — seller signals
- `isBuyable`, `isBestseller`, `stock`, `minOrderQty` — availability and order limits
- `stockWarning` — low-stock text such as `Only 2 left`
- `estimatedDelivery` — delivery promise on detail records
- `imageUrl`, `images` — primary image and full image list
- `promotionTags` — delivery, deal, and merchandising labels
- `dealTag`, `discountCode`, `bestSellerRank` — merchandising signals
- `description`, `featureBullets`, `specifications`, `breadcrumbs`, `categoryPath` — detail fields
- `warranty`, `offers`, `offerCount` — seller offers and warranty (detail mode)
- `market`, `locale` — storefront code and language-region
- `sourceQuery`, `sourceCategory` — the input that produced the row
- `scrapedAt` — ISO 8601 timestamp

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `search` | `search` / `byCategory` / `byProduct` / `byUrl` |
| `searchQueries` | array | – | Keywords (mode=search) |
| `categoryPaths` | array | – | Category paths or URLs (mode=byCategory) |
| `productInputs` | array | – | SKUs or product URLs (mode=byProduct) |
| `startUrls` | array | – | Search, category, or product URLs (mode=byUrl) |
| `market` | string | `uae` | `uae`, `saudi`, `egypt`, `bahrain`, `qatar`, `kuwait`, `oman` |
| `language` | string | `en` | `en` or `ar` |
| `sortBy` | string | `popularity` | `popularity`, `price_asc`, `price_desc`, `best_rating`, `new_arrivals` |
| `minPrice` / `maxPrice` | number | – | Price window in the market's currency |
| `minRating` | number | – | Minimum star rating (0–5) |
| `minDiscountPercent` / `maxDiscountPercent` | number | – | Discount window (0–100) |
| `brand` | string | – | Brand name or code, e.g. `HP` or `hp` |
| `seller` | string | – | Seller name or code, e.g. `noon` or `p_9404` |
| `dealOnly` | boolean | `false` | Only products with a noon deal tag |
| `includeDetails` | boolean | `false` | Add description, specs, stock, offers, warranty, delivery |
| `includeReviews` | boolean | `false` | Add up to five latest review snippets |
| `maxItems` | integer | `50` | Hard cap on emitted products (1–10,000) |

### Example: discounted laptops in the UAE

```json
{
  "mode": "search",
  "searchQueries": ["laptop"],
  "market": "uae",
  "minDiscountPercent": 20,
  "minRating": 4,
  "maxItems": 50
}
```

### Example: browse a Saudi category with product details

```json
{
  "mode": "byCategory",
  "categoryPaths": ["electronics-and-mobiles/computers-and-accessories/computers-new/laptops"],
  "market": "saudi",
  "sortBy": "price_asc",
  "maxPrice": 4000,
  "includeDetails": true,
  "maxItems": 100
}
```

### Example: look up products by SKU or URL in Egypt

```json
{
  "mode": "byProduct",
  "productInputs": [
    "N70430909V",
    "https://www.noon.com/egypt-en/renewed-hp-elitebook-845-g8-laptop-amd-ryzen-3-5450u-8gb-ddr4-ram-256gb-ssd-amd-radeon-tm-graphics-14-inch-fhd-windows11-pro-silver-english-arabic-silver/N70430909V/p/"
  ],
  "includeReviews": true,
  "maxItems": 10
}
```

### Example: paste noon URLs from any market

```json
{
  "mode": "byUrl",
  "startUrls": [
    "https://www.noon.com/uae-en/search?q=perfume&sort%5Bby%5D=price&sort%5Bdir%5D=asc",
    "https://www.noon.com/kuwait-en/electronics-and-mobiles/"
  ],
  "maxItems": 50
}
```

## Use cases

- **Discount monitoring** — track products above a discount threshold across GCC markets
- **Price intelligence** — compare current and list prices in each local currency
- **Seller and buy-box research** — see who sells a SKU, at what price, with what stock
- **Catalog enrichment** — pull titles, images, specs, and descriptions for product feeds
- **Market research** — compare assortment, availability, and ratings across seven storefronts
- **Customer-feedback sampling** — collect the latest review snippets and aggregate ratings

## FAQ

**Which noon markets are supported?**
UAE, Saudi Arabia, Egypt, Bahrain, Qatar, Kuwait, and Oman. Each returns prices in its own currency. English and Arabic storefronts are supported.

**Can I paste noon URLs instead of filling in fields?**
Yes. `byUrl` accepts search, category, and product URLs. The market and language in each URL override the Market and Language fields. URL price, rating, brand, and seller filters are applied when present; a deal filter in a URL becomes a deal-only filter.

**Why is `originalPrice` missing on some products?**
noon only exposes a list price when a product is discounted. Products without a discount omit `originalPrice`, `discountPercent`, and `discountAmount`.

**Why are rating or review count missing on some products?**
The values are omitted for products noon has not rated yet. Setting a positive `minRating` excludes unrated products.

**Can the actor return full review text?**
It returns up to the five latest review snippets through `includeReviews`. The full paginated review history is not publicly available to a cookieless request, so it is not claimed.

**How many products can one input return?**
Up to 10,000 per run. noon serves at most 10,000 rows per search or category input, 200 products per page with a page 50 cap. For larger catalogs, split the job by category, brand, or price window.

**Does the actor sort by biggest discount?**
noon does not provide a server-side discount sort. Use `minDiscountPercent` and `maxDiscountPercent` to target a discount range, then sort the dataset by `discountPercent` in your spreadsheet or database.

**How fresh is the data?**
Every run reads noon's live catalog responses. Prices, discounts, ratings, and stock reflect the storefront at the time of the run.

**Do I need a proxy or a noon account?**
No. The actor uses noon's public catalog data and handles its own connection automatically. No login, cookie, API key, or proxy configuration is required.

**Is this actor affiliated with noon?**
No. This is an independent third-party tool. It is not affiliated with or endorsed by Noon AD Holdings Ltd or noon.com.
