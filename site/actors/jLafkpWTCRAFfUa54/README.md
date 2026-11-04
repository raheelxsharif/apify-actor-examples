# Amazon Search, Category & Seller Listings Extractor

Pull Amazon product listings from **keyword search, category/search URLs, browse-node IDs, seller
storefronts, ASINs, and exact product URLs** in one run. Every record carries the listing card
data — price, brand, rating, badges, images, delivery hints, and organic/sponsored position — and
optional product-detail enrichment adds specifications, the full gallery, seller identity, rating
histogram, variants, and bestseller ranks. No login, cookies, or API key needed.

## What this actor does

- **Six additive input modes.** Combine `keywords`, `categoryUrls`, `categoryNodeIds`,
  `sellerIds`, `asins`, and `productUrls` in the same run; every source is processed and
  deduplicated by ASIN.
- **Exact product lookup.** ASIN and product-URL modes open `/dp/{ASIN}` directly. They never
  fall back to an Amazon keyword search, so a known product cannot turn into an unrelated result.
- **Real category listings.** Node IDs use Amazon's `/s?rh=n:{node}&fs=true` listing route, not
  the `/b?node=` merchandising landing page. Search, seller (`/s?me=`), and seller-profile
  (`/sp?seller=`) URLs are routed automatically.
- **Playwright reliability.** Each marketplace session is warmed up on the homepage and paced
  with human-like jitter. Continue-shopping interstitials, dog pages, rate limits, and
  interstitial challenges are classified and retried. Proxy selection is automatic: an available
  non-residential (datacenter/static) group in the marketplace's country, never residential by
  default.
- **Listing intelligence.** Brand, card image set and large image, organic position, sponsored
  flag, ships-to line, variant count, unit price, deal text, stock hint, sold-by line, and a
  genuine list price kept separate from Amazon's "Typical price".
- **Detail enrichment.** Specifications, gallery images, rating histogram, reviews link, seller
  ID/name/URL, variant dimensions, BSR, breadcrumbs, description, features, and stock status.
- **Safe output.** Empty fields are omitted at every depth; nothing is emitted as `null`.

## Output per product

### Listing fields (all modes)

- `asin`, `title`, `url`, `marketplace`, `sourceMode`, `searchQuery`
- `price` `{value, currency, display}`, `listPrice`, `typicalPrice`, `discountPercent`
- `stars`, `reviewsCount`, `brand`
- `thumbnailImage`, `images`, `imageLarge`
- `badge`, `coupon`, `dealText`, `stockText`, `unitPrice`, `soldByText`, `moreOptionsText`
- `boughtInPastMonth`, `boughtInPastMonthText`
- `deliveryEstimate`, `shipsTo`, `sustainabilityFeatures`, `variantsCount`
- `offerCount`, `lowestOfferPrice`
- `sponsored` / `isSponsored`, `resultType`, `prime` / `isPrime`
- `organicPosition`, `categoryPosition`
- `categoryUrl`, `categoryPage`, `sourceLayout`, `position`
- `resultsTotal`, `resultsFrom`, `resultsTo`, `facets`, `relatedSearches`
- `scrapedAt`

### Product-detail fields (detail enrichment or ASIN/product-URL modes)

- `brand`, `inStock`, `inStockText`, `currency`
- `specifications` (overview + technical tables)
- `images` (gallery), `imageLarge`
- `starsBreakdown` (share per star, 0.0-1.0)
- `reviewsLink`, `answeredQuestions`
- `breadCrumbs`, `description`, `features`
- `variantAsins`, `variantAttributes`, `priceVariants`
- `delivery`, `fastestDelivery`, `shippingPrice`, `returnPolicy`, `support`
- `seller` `{name, id, url}` — omitted when no real seller ID can be resolved
- `bestsellerRanks`, `locationText`

Fields that Amazon does not render for a specific product are omitted, never fabricated. This
includes render-dependent badges (Prime, sponsored), seller identity, and some buy-box details.

## Input

At least one of `categoryUrls`, `keywords`, `productUrls`, `asins`, `sellerIds`, or
`categoryNodeIds` is required. All modes are additive.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `categoryUrls` | array | – | Search, category, seller-profile, or product URLs; routed automatically |
| `keywords` | array | – | Search phrases; each builds `/s?k={query}` |
| `productUrls` | array | – | Exact product URLs; normalized to `/dp/{ASIN}` on their own marketplace |
| `asins` | array | – | Exact ASINs; each opens `/dp/{ASIN}` directly |
| `sellerIds` | array | – | Seller/merchant IDs; each builds `/s?me={id}` |
| `categoryNodeIds` | array | – | Browse-node IDs; each builds `/s?rh=n:{id}&fs=true` |
| `marketplaceDomain` | select | `amazon.com` | Marketplace for keyword/ASIN/seller/node modes (21 domains) |
| `maxItems` | integer | `0` | Global cap across all sources; `0` means unlimited |
| `maxItemsPerCategory` | integer | `10` | Cap per keyword/URL/seller/node source (1-10000) |
| `maxPages` | integer | `1` | Pages per listing source (1-50); keyword search itself stops at 7 |
| `scrapeProductDetails` | boolean | `true` | Visit each accepted product page for detail fields |
| `includeSpecifications` | boolean | `true` | Include the specifications object when detail pages are scraped |
| `sortBy` | select | `relevanceblender` | Featured, price, review, newest, or best-seller sort |
| `priceMin`, `priceMax` | number | – | Price range in the marketplace currency |
| `minRating` | number | – | Minimum star rating (0-5) |
| `includeSponsored` | boolean | `true` | Include sponsored placements |
| `onlyDeals` | boolean | `false` | Keep only cards with visible deal evidence |
| `primeOnly` | boolean | `false` | Keep only cards with a visible Prime badge |
| `rateLimitDelay` | integer | `2` | Base delay in seconds between requests (1-30) |

### Example: keyword search, cards only

```json
{
  "keywords": ["wireless earbuds", "bluetooth speaker"],
  "maxItemsPerCategory": 100,
  "maxPages": 3,
  "scrapeProductDetails": false,
  "sortBy": "review-rank",
  "minRating": 4
}
```

### Example: exact products and a product URL together

```json
{
  "asins": ["B08N5WRWNW", "B09X7MPX8L"],
  "productUrls": ["https://www.amazon.de/dp/B08N5WRWNW"],
  "scrapeProductDetails": true,
  "includeSpecifications": true
}
```

### Example: category node, seller storefront, and filters

```json
{
  "categoryNodeIds": ["172282"],
  "sellerIds": ["A1B2C3D4E5F6G7"],
  "marketplaceDomain": "amazon.co.uk",
  "maxItemsPerCategory": 50,
  "onlyDeals": true,
  "maxPages": 3
}
```

## Supported Amazon Marketplaces

amazon.com (US), amazon.co.uk (UK), amazon.de (Germany), amazon.fr (France), amazon.it (Italy),
amazon.es (Spain), amazon.ca (Canada), amazon.com.au (Australia), amazon.co.jp (Japan),
amazon.in (India), amazon.com.br (Brazil), amazon.com.mx (Mexico), amazon.nl (Netherlands),
amazon.se (Sweden), amazon.pl (Poland), amazon.com.tr (Turkey), amazon.ae (UAE),
amazon.sa (Saudi Arabia), amazon.sg (Singapore), amazon.com.be (Belgium), amazon.eg (Egypt).

## Use cases

- **Competitor price tracking** — monitor price, list price, typical price, and deal evidence.
- **Category and keyword research** — collect top products with brand, rating, and organic rank.
- **Sponsored-placement analysis** — compare organic position with sponsored rows.
- **Seller storefront audits** — enumerate a seller's listed products.
- **Product research** — pull full specifications, gallery, variants, and BSR for a shortlist.
- **Multi-marketplace checks** — compare the same ASIN or keyword across 21 storefronts.

## FAQ

**Do I need Amazon cookies or an API key?**
No. The actor reads public search, category, seller, and product pages anonymously. No account,
cookie, token, or API key is required.

**Do ASIN inputs still search Amazon?**
No. ASIN and product-URL modes fetch the exact `/dp/{ASIN}` page. Keyword mode is the only mode
that performs a search.

**Why is `listPrice` missing on a product that shows a "Typical price"?**
"Typical price" is not a strike-through list price. It is returned separately as `typicalPrice`,
and `discountPercent` is calculated only from a genuine current price plus list price. This
prevents fake discounts.

**Why are Prime or sponsored fields missing from some records?**
Amazon does not render Prime icons or sponsored labels for every marketplace, exit country, or
layout. The actor reports only what the page actually shows and never claims a product is "not
Prime" from a missing badge.

**Why do I get fewer results than `maxPages` suggests?**
Amazon caps keyword search at page 7. Category listings can go deeper. For more than a keyword
surface allows, split the query by category, brand, or price band and combine the runs.

**Does the actor need my proxy?**
No. Proxy selection is automatic: it uses an available non-residential Apify Proxy group in the
marketplace's own country and never switches to residential billing on its own. There is no proxy
input to configure.

**Why do prices appear in a currency different from the marketplace domain?**
Amazon localizes currency and delivery to the request's exit country. The actor sets the
marketplace country automatically and reports the resolved `currency`; cross-border renders can
still differ from the domain's nominal currency.

**How expensive is detail enrichment?**
Each detail page is a full Amazon product page (roughly 2.5-3.1 MB). Set
`scrapeProductDetails` to `false` for large listing runs and enable it only for a shortlist or
for ASIN/product-URL modes.

**Is this affiliated with Amazon?**
No. It is an independent third-party actor that reads Amazon's public pages.
