# 1188.lv Latvian Business & Registry Lookup

Pull structured business data from 1188.lv, Latvia's leading directory. Search any keyword or browse any of 384 business categories, and get full addresses, phone numbers, emails, websites, ratings, opening hours, social media links, photos, and official registry details for each result.

## What this actor does

- **Three modes:** `search` (site-wide keyword search), `byCategory` (browse one of 384 real 1188.lv categories), and `byUrls` (fetch specific businesses by direct URL)
- **All 384 categories** available as a dropdown, or type any custom `slug-id` value
- **City filter** — restrict results to any Latvian city or town by matching the parsed address
- **Filter by** minimum star rating (1-5), "has website", "has email"
- **Rich business profiles** — every listing is enriched with its full detail page: phone number(s), email(s), website, GPS coordinates, seasonal opening hours, social media links, photo gallery, product/service keyword tags, and full legal registration details
- **Full Latvian character support** — ā, č, ē, ģ, ī, ķ, ļ, ņ, š, ū, ž are handled correctly in both input and output
- **Empty fields are omitted** — you only ever see fields that were actually found for a business

## Output per business

- `businessId` — 1188.lv internal business ID (`search`/`byCategory` modes; not set in `byUrls` mode)
- `name` — business name
- `category` — the business's own category/industry label(s) as published on its detail page (e.g. "Restorāns"), available in every mode including `byUrls`
- `description` — business description snippet
- `rating` (1-5), `reviewCount` — from the business's own detail-page rating block (most current/accurate)
- `fullAddress` — the complete address as published; `street`, `city`, `zipCode` — best-effort parsed parts
- `latitude`, `longitude` — precise GPS coordinates
- `phone` — primary phone number, `allPhones[]` — every phone number listed (businesses often list several)
- `email` — primary email address, `allEmails[]` — every email listed (1188.lv's own contact address is always excluded)
- `website` — business website (1188.lv's own UTM tracking parameters stripped)
- `openingHours[]` — structured opening hours (`{day, hours, season}` per slot; `season` is set only for businesses that publish separate winter/summer hours)
- `socialLinks` — business's own social media profiles, keyed by platform (e.g. `facebook`, `instagram`, `tripadvisor`)
- `photos[]` — photo gallery image URLs published on the business's profile
- `keywords[]` — product/service keyword tags the business is listed under
- `serviceArea` — the geographic area a business explicitly states it serves (delivery/service businesses), when published
- `registrationNumber`, `registrationDate`, `registerName` — Latvian business registry number, registration date, and register name (e.g. "Komercreģistrs"), when published
- `registrationStatus` — the business's current registry status (e.g. "Reģistrēts"), when published
- `legalForm` — legal form of the business (e.g. "Sabiedrība ar ierobežotu atbildību"), when published
- `legalAddress` — the business's registered legal address, when published (may differ from its operating `fullAddress`)
- `logoUrl` — business logo/photo
- `sourceUrl` — canonical 1188.lv business page URL
- `recordType: "business"`, `scrapedAt`

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string (select) | `search` | `search` (keyword), `byCategory` (browse), or `byUrls` (fetch by URL) |
| `query` | string | `restorāns` | Keyword to search for (mode=search) |
| `category` | string (select) | `restorans-2686` | Pick from all 384 real 1188.lv categories, or `custom` to use `customCategory` (mode=byCategory) |
| `customCategory` | string | – | Any `slug-id` value, overrides `category` (mode=byCategory) |
| `city` | string | – | Only return businesses whose address contains this city/town (any mode) |
| `businessUrls` | array | – | 1188.lv business page URLs to fetch directly (mode=byUrls) |
| `minRating` | integer | `0` (no minimum) | Only return businesses rated at least 1-5 stars |
| `requireWebsite` | boolean | `false` | Only return businesses with a listed website |
| `requireEmail` | boolean | `false` | Only return businesses with a listed email address |
| `maxItems` | int | `20` | Hard cap on returned records (1-1000) |

### Example: search for restaurants site-wide

```json
{
  "mode": "search",
  "query": "restorāns",
  "maxItems": 20
}
```

### Example: browse dentists in Rīga with at least a 4-star rating

```json
{
  "mode": "byCategory",
  "category": "zobarstnieciba-6610",
  "city": "Rīga",
  "minRating": 4,
  "maxItems": 30
}
```

### Example: banks with a published website and email

```json
{
  "mode": "byCategory",
  "category": "bankas-890",
  "requireWebsite": true,
  "requireEmail": true,
  "maxItems": 50
}
```

### Example: fetch specific businesses by URL

```json
{
  "mode": "byUrls",
  "businessUrls": [
    { "url": "https://www.1188.lv/katalogs/restorans-2686/armenu-virtuve-nojan-tapan-498662" }
  ]
}
```

## Use cases

- **Local lead generation** — build a prospect list of Latvian businesses by category and city for B2B outreach
- **Market research** — measure business density and average ratings for a category across Latvian regions
- **Directory aggregation** — feed structured Latvian business data into your own local-search product
- **Competitor mapping** — see who else operates in a category and area, with ratings and contact info
- **Registry cross-referencing** — match businesses to Latvia's official registration numbers

## Limitations

- **`city` is a client-side address filter, not a server-side one.** 1188.lv's own location filter (`vzd_code`) only lists cities that already have results within the specific category/search being browsed, so there's no single stable city list valid across all categories. Instead, `city` matches the parsed address text (diacritic- and case-insensitive), which works uniformly everywhere but depends on the city name appearing in the address as published.
- **No authoritative total-result count.** 1188.lv's server-rendered listing pages don't expose a total-hit count, so pagination stops on the site's own natural signal (two consecutive empty pages) rather than a known total.
- **One extra request per result.** Every listing result is automatically enriched with its detail page (phone/email/website/hours only exist there, not on listing cards), so `byCategory`/`search` modes are roughly twice as slow per record as a listing-only scrape — this is by design, to guarantee complete records.

## FAQ

**What data source does this actor use?**
It reads the publicly available search, category-browse, and business-detail pages on 1188.lv, Latvia's leading business directory. No account or API key is needed.

**Is this affiliated with 1188.lv?**
No. This is an independent, third-party actor that reads publicly accessible pages.

**Why do some businesses have no rating, phone, or website?**
1188.lv only shows a rating once a business has received at least one review, and only shows contact details a business itself provided. Fields with no data are simply omitted from the record rather than shown as empty or null.

**What's the difference between `phone` and `allPhones`?**
`phone` is the primary number (from the business's structured data). `allPhones` additionally appears only when the business lists more than one number, and includes every number found on its detail page.

**Does this actor need a proxy or login?**
No — 1188.lv's search, category, and detail pages are served without any anti-bot challenge, so this actor runs entirely on Apify's free plan with no proxy and no credentials.

**How do I find a category's `slug-id` value?**
Browse https://www.1188.lv/katalogs and copy the URL segment after `/katalogs/` for any category (e.g. `restorans-2686` for restaurants), or just pick it from the `category` dropdown.
