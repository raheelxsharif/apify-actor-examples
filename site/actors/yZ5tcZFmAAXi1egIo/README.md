# Amazon Deals Grid Tracker by Category & Discount

Pull Amazon's "Today's Deals" grid across 23 marketplaces and get every discounted product's ASIN, title, images, brand, department/category, variation options, and pinned status back as a structured record. Filter by category, brand, price range, discount range, and deal type (see FAQ for the current status of the `minRating` filter). No login required.

## What this actor does

- **23 marketplaces** — amazon.com, .co.uk, .de, .fr, .it, .es, .ca, .com.au, .co.jp, .in, .com.br, .com.mx, .nl, .se, .pl, .com.tr, .ae, .sa, .sg, .eg, .ie, .com.be, .co.za
- **Filters:** category/department, brand, deal price range, discount percent range, deal type (real collection bubbles Amazon shows on the page); a `minRating` input also exists but currently has no effect since Amazon's Deals grid never exposes per-deal ratings (see FAQ)
- **Deal fields:** title, images, deal/list price, savings amount/percent, deal badge text/messaging, countdown-badge expiry timestamp, claim progress, deal state/promotion type, lightning-deal flag, brand name/ID, department names/IDs, category, variation options, pinned flag
- **Pagination:** walks the deals grid page by page up to your limits
- **Empty fields are omitted**

## Output per deal

- `asin` — Amazon Standard Identification Number
- `title` — product title
- `sourceUrl` — canonical product URL
- `images[]` — product image URLs (hi-res and low-res variants)
- `dealPrice`, `listPrice` — `{value, currency, display}` — included whenever Amazon's Deals grid carries live pricing for that specific deal (a real subset of deals — typically roughly half on any given fetch — genuinely have no price data attached, e.g. brand-level coupon promos; never fabricated, see FAQ)
- `savingsAmount`, `savingsPercent` — included whenever Amazon exposes them (derived from the deal's own price fields and/or its badge percentage, whichever is present)
- `dealBadgeText` — the deal grid's own badge label (e.g. `"15% off"`), when shown
- `dealBadgeMessage` — the badge's messaging line (e.g. `"Limited time deal"`), when shown
- `dealEndsAt` — ISO-8601 UTC expiry timestamp for a Lightning-Deal-style countdown badge (e.g. `"Ends in"`), when Amazon exposes one for that deal
- `percentClaimed`, `percentClaimedMessage` — live claim-progress for the deal (e.g. `34.0`, `"34% claimed"`), when Amazon exposes it for that deal
- `dealState` — the deal's live state as Amazon reports it (e.g. `AVAILABLE`)
- `dealPromotionType` — Amazon's internal promotion classification for the deal (e.g. `BEST_DEAL`, `LIGHTNING_DEAL`)
- `isLightningDeal` — boolean, whether Amazon's own cart data flags this as a Lightning Deal
- `brandId`, `brandName` — Amazon's internal brand ID and its resolved display name
- `departmentIds[]`, `departmentNames[]` — Amazon's internal department/category IDs and resolved names
- `categorySymbol`, `productType` — Amazon's internal category taxonomy codes
- `brandLogoUrl` — the brand's logo image, when shown
- `variationDimensions` — variant/size/color options and their ASINs (when present)
- `isPinned` — whether Amazon pinned this deal
- `dealId` — stable deal/promotion ID (when exposed by the widget)
- `position` — 1-based rank in the deals grid
- `recordType: "deal"`, `scrapedAt`

### When no deals are emitted

If a run ends with zero `"deal"` records, it also pushes exactly one `recordType: "status"` record instead of leaving the dataset empty with no explanation — so you can tell a real failure apart from a clean run that legitimately found nothing, without needing to read the run log:

- `success: false` — the Deals grid was never reached, with a `reason` of `BLOCKED` (couldn't get past bot-detection/redirect handling after retries — check `classification` for the specific cause), `WIDGET_NOT_FOUND` (page fetched cleanly but the deals widget wasn't present), or `NO_ENTRIES` (widget found but empty)
- `success: true, reason: "NO_MATCHES"` — the Deals grid was fetched and parsed successfully, but every entry was filtered out by your `category`/`brand`/`priceMin`-`priceMax`/`discountMin`-`discountMax`/`dealType` filters (or the marketplace's grid was genuinely empty that day)

Every status record also carries `marketplace`, `message` (human-readable), and `scrapedAt`.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `marketplace` | string | `amazon.com` | Which Amazon domain's Deals page to scrape |
| `dealType` | string | `(any)` | Real collection bubble Amazon shows on the page — Lightning Deals / Coupons / Outlet / Customers' Most-Loved / Lowest Price in 365 Days / plus 30+ per-category bubbles (Electronics, Beauty, Fashion, Home, Toys & Games, etc.), all live-verified against amazon.com/deals |
| `category` | string | – | Free-text filter matched against department name / category taxonomy code |
| `brand` | string | – | Free-text brand substring filter |
| `minRating` | int | – | Minimum star rating (1-5) — currently a no-op; see FAQ |
| `priceMin` / `priceMax` | int | – | Deal price range (local currency, when price data is available — see FAQ) |
| `discountMin` / `discountMax` | int | – | Discount percent range (when available — see FAQ) |
| `maxItems` | int | `30` | Hard cap on emitted deals (1-2000) |
| `maxPages` | int | `5` | Hard cap on grid pages walked (1-50) |
| `proxyConfiguration` | object | AUTO | Apify proxy configuration for fetching the Deals page |

### Example: default US deals grid

```json
{
  "marketplace": "amazon.com",
  "maxItems": 30
}
```

### Example: home & kitchen deals from a specific brand

```json
{
  "marketplace": "amazon.com",
  "category": "Home & Kitchen",
  "brand": "Bissell",
  "maxItems": 60
}
```

### Example: UK Customers' Most-Loved deals with a 4-star minimum

```json
{
  "marketplace": "amazon.co.uk",
  "dealType": "deals-collection-cml",
  "minRating": 4,
  "maxItems": 50
}
```

## Use cases

- **Deal aggregators** — build a curated "today's deals" catalog feed
- **Competitive intelligence** — see which brands/categories Amazon is currently discounting
- **Merchandising research** — track which products get pinned/featured in the Deals grid
- **Shopping assistants** — surface deals matching a user's category/brand interest
- **Catalog enrichment** — pull variation options (size/color) and brand metadata for deal ASINs

## FAQ

**Does this require an Amazon account or cookies?**  No. The Deals page is publicly browsable; the actor scrapes it directly with no login.

**Why don't I see `dealPrice`, `listPrice`, `savingsPercent`, `dealBadgeText` or `percentClaimed` on every record?**  Amazon's own Deals grid data genuinely omits price/badge/claim-progress information for a real subset of deals on any given fetch (commonly brand-level coupon promos with no single "deal price" to show). The actor never fabricates these fields — it populates them whenever Amazon's page includes them for that specific deal, and omits them entirely otherwise. All the catalog/structural fields (title, images, brand, category, variations) are always populated when Amazon shows them.

**Why do a few records occasionally show an implausibly large `dealPrice`/`listPrice`?**  On rare fetches, Amazon's own Deals grid data itself carries an inflated numeric value for a handful of deals — independently confirmed live by inspecting the raw page data byte-for-byte, not an actor parsing/arithmetic error (the actor's `savingsPercent` math on these records is still internally correct — e.g. a genuinely-computed "30% off" — only the absolute base price is implausible). This appears tied to sessions Amazon treats as higher-risk/automated (the same class of session-dependent behavior that can also cause Amazon to render a different regional currency than expected); the actor does not attempt to "correct" these values since there is no reliable way to distinguish a genuinely expensive real deal from an inflated one without fabricating a number. If you filter downstream by price, treat an outlier far outside a category's normal range with appropriate skepticism.

**Why is `dealType` filtering only "best effort" for some values?**  Amazon's Deals page groups deals into filter "bubbles". Category-style bubbles (e.g. Customers' Most-Loved) are backed by an explicit department list, so the actor can verify membership per deal. Promotion-style bubbles (Lightning Deals, Coupons, Outlet) aren't tagged per-item in the grid data the actor can see, so those filters pass every deal through rather than risk wrongly excluding a real match. Additionally, not every `dealType` value is guaranteed to render as a live bubble on every one of the 23 supported marketplaces on a given day (e.g. a smaller marketplace's Deals page may simply not have a "Jewelry" bubble today) — when this happens the actor logs a warning and passes every deal through unfiltered rather than silently returning zero results; check the run log if a `dealType` filter appears to have had no effect.

**Why does `sourceUrl` return a 503 if I fetch it with a plain script?**  Amazon aggressively blocks plain, non-browser requests — a bare request with no session or JavaScript can get a 503 "Service Unavailable" even though the same URL loads fine for a real shopper's browser. The URLs in the output are the correct, canonical Amazon product URLs and render normally when opened in an actual browser.

**Does `minRating` actually filter anything?**  Not currently, in practice. Amazon's Deals grid data (the same page payload documented above) never includes a per-deal star rating — only structural/catalog fields do. Since the actor never excludes a deal on the *absence* of rating evidence (per its no-fabrication rule), `minRating` currently passes every deal through unfiltered rather than risk wrongly dropping real matches. The input is kept because a future/differently-trusted session could expose rating data the way it can expose pricing; if you need rating-based filtering today, filter the output dataset by `stars` after the run (it will simply be absent on every current record).

**Do `priceMin`/`priceMax`/`discountMin`/`discountMax` exclude deals with missing price/savings data, or let them through?**  They let them through. Every filter in this actor follows the same rule: a deal is only ever *excluded* when it has real, present evidence that contradicts the filter — never on the *absence* of the field the filter checks (see "Why don't I see `dealPrice`..." above; the same real subset of deals genuinely has no price/savings data on any given fetch). So `discountMin: 10` will exclude a deal it can see has e.g. a 5% discount, but will still include a deal with no `savingsPercent` at all — its real discount is simply unknown, not verified to be below your threshold. If your workflow needs a hard guarantee ("only deals I can *confirm* meet the discount floor"), filter the output dataset post-run by checking `savingsPercent` is present and satisfies your threshold yourself, rather than relying on the actor's own filter to have done that exclusion for you.

**How many deals can I get per run?**  Amazon's Deals grid returns up to 30 items per page (occasionally fewer, depending on catalog/session state). The actor attempts to walk further pages by reproducing the grid's own "load more" request live in-browser; if Amazon doesn't expose a further page in a given session, the actor stops cleanly rather than guessing. Each deal's `position` is always a gapless, 1-based rank across the full run regardless of how many items any individual page actually returned.

**What if my category/brand filter returns 0 results?**  These are free-text substring filters matched against the department name / category code / brand name Amazon exposes on each deal. If nothing matches, widen or clear the filter.

**Is this affiliated with Amazon?**  No, this is an independent third-party actor that scrapes Amazon's public Deals page.

**How fresh is the data?**  As fresh as the live page at run time — Amazon refreshes its Deals grid continuously throughout the day.
