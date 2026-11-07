# Amazon Brand Store Page & Merchandising Crawler

Brands design their Amazon Store page like a mini storefront — navigation tabs, banner creative, merchandising rows, and a curated product mix. This actor crawls that storefront, walking the navigation menu into subpages, and returns its structure at every level: pages, sections, products, and links.

## What this actor does

- Crawls a Brand Store's landing page and, optionally, the subpages reachable through its own navigation menu.
- Resolves a brand's Store URL automatically when you only have a product page URL or ASIN instead.
- Emits one record per store page, one per merchandising section, one per distinct product, and one per unique target link inside each section.
- Captures each section's widget type, position, heading, body copy, call-to-action, primary image/video, and product/link counts.
- Tags every product and link with the section it came from, so a merchandising audit can be grouped or filtered without re-parsing the page.
- Reports a page explicitly as blocked or missing rather than silently returning it as an empty store.

A Brand Store is curated by the brand, not a full product catalog — Amazon doesn't guarantee every item a brand sells appears there. This actor reports every product link it can find on the pages it crawls; for a seller's complete catalog, a seller-shop scraper is the better tool.

## Output: store page record (recordType: storePage)

| Field | Description |
|---|---|
| `storeUrl` | The Brand Store URL this page belongs to |
| `brand` | Brand name |
| `storePageId` | Page identifier from the URL |
| `pageName` | Navigation label for this page, e.g. "HOME", "FEATURED DEALS" |
| `description` | Page meta description, when present |
| `logoUrl` | Brand logo image |
| `navigation` | Subpage links found on this page, each with `name`, `url`, `storePageId`, `order` (populated when `crawlSubpages` is on) |
| `sectionCount` | Merchandising sections found on this page |
| `productCount` | Distinct products linked from this page |
| `linkCount` | Unique links found on this page |
| `depth` | How many navigation hops from the landing page this page is |
| `pagePosition` | Position in the crawl order |
| `scrapedAt` | When the actor collected the record |

## Output: merchandising section record (recordType: section, when includeSections is on)

| Field | Description |
|---|---|
| `storeUrl`, `brand`, `pageName`, `storePageId` | Which store and page the section appears on |
| `sectionType` | Widget type, e.g. `Header`, `EditorialRow`, `MultiMediaCarousel`, `Hero` |
| `sectionId` | Amazon's widget ID for this section |
| `sectionPosition` | Position of the section in page order |
| `heading`, `bodyText` | Section title and body copy, when present |
| `ctaText` | Section call-to-action text, e.g. "Shop now", "Learn more" |
| `imageUrl`, `imageAlt`, `imageCount` | Primary section image and how many section-level images it contains |
| `videoUrl`, `videoThumbnail`, `videoTitle`, `videoCount` | Video module fields |
| `destinationUrl` | Primary non-product link target of the section |
| `productCount`, `linkCount` | Distinct products and unique links inside the section |
| `depth` | Navigation depth of the page |
| `scrapedAt` | When the actor collected the record |

## Output: product record (recordType: product)

| Field | Description |
|---|---|
| `asin` | Amazon product ID |
| `title` | Product title, when visible next to the link |
| `url` | Product page URL |
| `thumbnailImage` | Product thumbnail |
| `price`, `listPrice` | Price and list price, each with `value`, `currency`, `display`, only when visible on the page |
| `stars`, `reviewsCount` | Rating and review count, only when visible near the link |
| `badge` | One of `BestSeller`, `AmazonsChoice`, `OverallPick`, `LimitedTimeDeal`, `NewOnAmazon`, `SmallBusiness`, only when shown |
| `discountPercent` | Percent off, only when a discount badge is shown |
| `brand` | Brand name |
| `storeUrl`, `storeSubPage` | Store and page the product was found on |
| `sectionType`, `sectionId`, `sectionPosition` | The merchandising section that contains the product tile |
| `modulePosition` | Position of the product tile on the page |
| `scrapedAt` | When the actor collected the record |

## Output: link record (recordType: link, when includeLinks is on)

| Field | Description |
|---|---|
| `storeUrl`, `brand`, `pageName`, `storePageId` | Which store and page the link appears on |
| `linkType` | `product`, `storePage`, `search`, `external`, or `other` |
| `url` | Absolute target URL, with store-page tracking parameters removed |
| `text`, `ariaLabel` | Visible link text and accessibility label |
| `asin` | Product ID for product links |
| `destinationStorePageId` | Target page ID for store-page links |
| `isExternal` | Present and `true` for links that leave the store's Amazon domain |
| `sectionType`, `sectionId`, `sectionPosition` | The section the link appears in |
| `linkPosition` | Position of the link in page order |
| `depth` | Navigation depth of the page |
| `scrapedAt` | When the actor collected the record |

## Output: page error record (recordType: storePageError)

Emitted instead of a `storePage` record when a page couldn't be read. Fields: `storeUrl`, `pageClassification` (e.g. `NOT_FOUND`, `CAPTCHA`, `AUTH_REQUIRED`), `depth`, `pagePosition`.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `storeUrls` | Array of strings | `[]` (prefill: soundcore landing page) | Brand Store URLs to crawl. At least one of `storeUrls`, `productUrls`, or `asins` is required. |
| `productUrls` | Array of strings | – | Product page URLs; the actor finds that product's "Visit the Store" link and crawls the store instead. |
| `asins` | Array of strings | – | ASINs on amazon.com with the same auto-resolve behavior as `productUrls`. |
| `crawlSubpages` | Boolean | `true` | Follow the store's own navigation menu into subpages. |
| `maxDepth` | Integer | `1` | How many navigation hops to follow from the landing page (1–5). |
| `maxPages` | Integer | `10` | Maximum store pages fetched per store, landing page included (1–50). |
| `maxProducts` | Integer | `200` | Maximum distinct product records emitted per store (1–2000). |
| `includeSections` | Boolean | `true` | Emit a `section` record for every merchandising module. |
| `includeLinks` | Boolean | `true` | Emit a `link` record for every unique on-page link. |
| `proxyConfiguration` | Proxy | Apify Proxy | Used for every request; automatically escalates to a different proxy tier if the store keeps blocking requests. |

### Example: crawl a Brand Store including its subpages

```json
{
  "storeUrls": ["https://www.amazon.com/stores/soundcore/page/AD6AD8C0-90C7-4C65-8000-0AC741FA2633"],
  "crawlSubpages": true,
  "maxDepth": 2,
  "maxPages": 15,
  "maxProducts": 100
}
```

### Example: landing page only, pages and products (no sections or links)

```json
{
  "storeUrls": ["https://www.amazon.com/stores/soundcore/page/AD6AD8C0-90C7-4C65-8000-0AC741FA2633"],
  "crawlSubpages": false,
  "includeSections": false,
  "includeLinks": false
}
```

### Example: resolving a store from a product page

```json
{
  "storeUrls": [],
  "productUrls": ["https://www.amazon.com/dp/B09X7MPX8L"]
}
```

## Use cases

- **Brand monitoring teams** track how a competitor merchandises and navigates their Amazon storefront.
- **Merchandising researchers** compare section types, headings, CTAs, and featured-product mix across brands.
- **Catalog discovery teams** find every product a brand currently features on its Store page.
- **Creative audit teams** collect a brand's storefront imagery, video, and CTA copy in one pass.
- **Link-integrity teams** export every navigation, store-page, product, and external target in one dataset.

## FAQ

**What is the difference between a section and a link?**
A section is a merchandising module (one widget on the page). Links are the individual targets inside those sections. A target that appears in more than one section gets one link record per section, so `linkCount` on a page can be lower than the number of link records. Both carry `sectionType`/`sectionId`, so they join cleanly.

**Why don't I see everything behind a "See products" modal?**
Shoppable-image modals load their product list only after a click. The actor reads the products already rendered on the page (carousels, grids, editorial tiles) and does not click through modals.

**My saved task sends `includeAssets` — will it still work?**
Yes. The old key is still accepted and mapped in code. Tasks that send it keep receiving the original flat `asset` records; new tasks should use `includeSections` and `includeLinks`.

**Is this the same as a seller's product catalog?**
No. A Brand Store is a curated page the brand designs; a seller catalog lists everything that seller has for sale. Use this actor for the storefront experience, and a seller-shop tool for the full catalog.

**Why do some products have no price or rating?**
Brand Store product tiles don't always render price or rating in the page itself — a field is only included when it's actually visible next to the product link, never a guessed value.

**What if I only have a product URL, not a store URL?**
Pass it via `productUrls` (or the ASIN via `asins`); the actor opens that product page, follows its "Visit the Store" link, and crawls that store instead.

**Does this need a login or cookies?**
No. Brand Store pages are public.

**Why does the actor sometimes report a `storePageError` instead of products?**
Amazon occasionally serves a block or rate-limit page instead of the real store. Rather than reporting zero products as if the store were simply empty, the actor records explicitly what happened on that page.

**Does this capture every product a brand sells?**
No — see "What this actor does" above; a Brand Store is editorially curated, not a full catalog.

**Which Amazon marketplaces are supported?**
Any Amazon domain — amazon.com, amazon.co.uk, amazon.de, and so on — just pass the full Store URL for that marketplace.

**Is this affiliated with Amazon?**
No. This actor independently reads Amazon's public Brand Store pages and is not affiliated with or endorsed by Amazon.
