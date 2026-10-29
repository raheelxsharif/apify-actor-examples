# YellowPagesPK Business Directory & Lead Finder

YellowPagesPK.com lists Pakistani businesses across dozens of categories and cities. This actor browses any of the site's 46 business categories, runs a free-text keyword search (business name, service, or city), or fetches full business profiles from direct listing URLs — names, addresses, cities, phone numbers, emails, websites, and more, with no login, cookies, or API key required.

## What this actor does

- **Three modes:** `byCategory` (browse a category, with real pagination), `search` (free-text keyword search across the whole directory, with real pagination), and `byUrls` (fetch specific listing pages directly)
- **46 categories** covering restaurants, hotels, automotive, real estate, law firms, medical, education, construction, and more — the full, current category list published on the site
- **Optional city filter** — narrow any mode to businesses whose address mentions a given Pakistani city (e.g. `"Lahore"`, `"Karachi"`, `"Islamabad"`)
- **Full business profiles** — description, category, claimed status, owner/author profile, email, phone, website, logo (toggle `fetchDetails`)
- **Empty fields are omitted** — you only ever see fields that were actually found for a business

## Output per business

- `name` — business name
- `description` — business description (when profile fetched)
- `category`, `categorySlug` — the listing's own category, taken from its breadcrumb (when profile fetched)
- `searchCategory` — the category you searched (mode=byCategory)
- `searchQuery` — the keyword you searched (mode=search)
- `address` — full street address as published
- `city` — best-effort city extracted from the listing's own city tag (mode=byCategory) or from the address text
- `phone`, `email`, `website`
- `latitude`, `longitude` — GPS coordinates (only present for the small number of listings that publish them)
- `rating` — star rating out of 5 (only present for listings that have received at least one review — most have none)
- `reviewCount` — number of reviews behind the rating (mode=byCategory; only present when `rating` is present)
- `openStatus` — live open/closed/day-off badge text, when the listing publishes hours (mode=byCategory)
- `claimed` — `true` if the business owner has claimed and verified the listing
- `sponsored` — `true` if the listing is marked "Ad" (a paid/promoted placement) on its category page (mode=byCategory)
- `imageUrl` — listing photo (mode=byCategory; omitted when the listing has no real photo, or the photo file is no longer reachable on the source site -- see Limitations)
- `logoUrl` — business logo (when profile fetched; omitted when the listing has no real logo, or the logo file is no longer reachable on the source site -- see Limitations)
- `authorName`, `authorProfileUrl` — the directory account that submitted the listing (when profile fetched)
- `sourceUrl` — canonical YellowPagesPK.com listing URL
- `recordType: "business"`, `scrapedAt`

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string (select) | `byCategory` | `byCategory`, `search`, or `byUrls` |
| `category` | string (select) | `restaurant` | One of the 46 published categories (mode=byCategory) |
| `cityFilter` | string | – | Only keep businesses whose address/city mentions this text, e.g. `"Lahore"` (mode=byCategory, mode=search) |
| `searchQuery` | string | – | Free-text keyword to search, e.g. `"restaurant lahore"` (mode=search) |
| `listingUrls` | array | – | Direct YellowPagesPK.com listing URLs to fetch (mode=byUrls) |
| `minRating` | integer (select) | `0` | Only include businesses rated at least this many stars (0-5). Businesses with no published rating are excluded once set above 0 |
| `onlyClaimed` | boolean | `false` | Only include businesses whose listing has been claimed/verified by its owner |
| `fetchDetails` | boolean | `true` | Also visit each listing's page for description, email, website, category, claimed status, and owner profile |
| `maxItems` | integer | `30` | Hard cap on returned records (1-500) |

### Example: browse restaurants

```json
{
  "mode": "byCategory",
  "category": "restaurant",
  "maxItems": 30
}
```

### Example: hotels in Lahore only

```json
{
  "mode": "byCategory",
  "category": "hotels",
  "cityFilter": "Lahore",
  "maxItems": 30
}
```

### Example: keyword search across the directory

```json
{
  "mode": "search",
  "searchQuery": "digital marketing lahore",
  "maxItems": 40
}
```

### Example: fetch specific business profiles

```json
{
  "mode": "byUrls",
  "listingUrls": [
    { "url": "https://yellowpagespk.com/listing/andaaz-restaurant/" }
  ]
}
```

### Example: only claimed/verified listings

```json
{
  "mode": "byCategory",
  "category": "law-firms",
  "onlyClaimed": true,
  "maxItems": 30
}
```

## Use cases

- **Local lead generation** — build a prospect list of Pakistani businesses by category and city for B2B outreach
- **Directory aggregation** — feed structured Pakistani business data into your own local-search product
- **Market research** — see how many listed businesses exist in a category and which cities they cluster in
- **Competitor mapping** — see who else is listed in a category with contact and website details

## Limitations

- **`mode=byCategory` paginates for real.** YellowPagesPK.com's category pages (`/listing-category/<slug>/`, `/listing-category/<slug>/page/N/`) are genuinely paginated via standard WordPress `/page/N/` URLs (confirmed live: page 1 and page 2 return distinct, non-overlapping business listings). The actor walks pages until it hits `maxItems`, runs out of new listings, or reaches an internal page cap.
- **`mode=search` also paginates for real** and can return substantially more results than `mode=byCategory` for a broad keyword, since it uses the site's native WordPress search (`/?s=...`, `/page/N/?s=...`), which is genuinely paginated.
- **Ratings and reviews are rare.** Most listings on this directory have never received a review, so `rating` is only present on the small minority that have. This is a fact about the underlying data, not a bug — the actor never fabricates a rating.
- **Many older listing photos are broken on the source site itself.** YellowPagesPK.com's HTML still references photo/logo files uploaded before roughly November 2024 that have since been removed from its media server (verified: the site's own listing pages link to these paths, but the files themselves return 404). The actor live-checks every `imageUrl`/`logoUrl` before including it and omits the field rather than shipping a broken link, so a lower `imageUrl`/`logoUrl` coverage rate reflects an upstream data-integrity gap, not a scraper defect.
- **GPS coordinates are rare.** Only a small number of listings have `latitude`/`longitude` set on the site itself; most do not, and the field is simply omitted.
- **`city` is best-effort.** `mode=byCategory` uses the listing's own city tag when the site provides one; otherwise (and always in `mode=search`/`mode=byUrls`), the actor matches the address text against a curated list of ~70 major Pakistani cities/towns. Addresses that don't mention a recognized city will have no `city` field.
- **`cityFilter` only excludes businesses with a known, non-matching city.** Businesses that have no address/city data at all on the site are passed through rather than excluded (the filter can't verify what isn't there), the same way `minRating`-style filters behave on other directory actors in this catalog.
- **Directory data quality is the site's own.** YellowPagesPK.com occasionally contains a small number of off-topic or foreign listings inside a Pakistani category (a fact of the source data, reproduced as-is rather than filtered out, since the actor does not assume what "off-topic" means for every use case).

## FAQ

**What data source does this actor use?**
It reads the publicly available category, search, and listing pages on YellowPagesPK.com, a Pakistan business directory. No account or API key is needed.

**Is this affiliated with YellowPagesPK.com?**
No. This is an independent, third-party actor that reads publicly accessible pages.

**Why do some businesses have no email, website, or rating?**
YellowPagesPK.com only shows contact details a business itself provided, and only shows a rating once a business has received at least one review. Fields with no data are simply omitted from the record rather than shown as empty or null.

**Why is `fetchDetails` on by default?**
Category and search listing pages alone only carry a subset of fields (name, address, phone, sometimes a photo). Turning on `fetchDetails` (the default) visits each listing's own page for the full picture — description, category, email, website, claimed status, and owner profile — at the cost of one extra request per result.

**Can I search by both category and city at once?**
Yes — set `category` and `cityFilter` together in `mode=byCategory` to browse a category limited to a specific city.

**Does this actor need a proxy or login?**
No. YellowPagesPK.com's category, search, and listing pages are publicly accessible, so this actor runs with no login and no credentials required.
