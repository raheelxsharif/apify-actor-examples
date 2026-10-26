# Google Maps Business Lead & Email Finder

Turn a Google Maps search into a ready-to-use list of sales and marketing leads. Each result carries the place's own contact details, and when enrichment is on, the business's own website is checked for an email address and social profiles. No Google account, cookies, or API key are needed to run a search.

## What this actor does

- Search Google Maps by any query ("restaurants in Brooklyn", "dentists Berlin")
- Extract name, category, address, phone, website, rating, reviews, place ID, coordinates
- Optional website crawl: emails, additional phones, Facebook / Instagram / LinkedIn / X / YouTube / TikTok
- Supports the country and language values listed in the input schema; raw Maps queries can target other locations as well

## Input

| Field | Type | Default | Description |
|------|------|---------|-------------|
| `category` | string | – | Business category; used only when `keyword` and `searchQueries` are empty |
| `keyword` | string | – | Custom search term; takes precedence over `category` |
| `state` | enum | `CA` | US state code or `All`; used for US searches |
| `country` | enum | – | Country code: `US`, `GB`, `CA`, `AU`, `DE`, `FR`, `IN`, `BR`, `MX`, `JP`, `KR`, `CN`, `AE`, or `SG` |
| `max_results` | int | 100 | Total lead cap (1–100,000; records are pushed progressively) |
| `minRating` | number | – | Optional minimum rating from 1.0 to 5.0 |
| `extractEmails` | bool | `true` | Crawl public business websites for emails, additional phones, contact pages, and social links |
| `language` | enum | `en` | Maps interface language: `en`, `es`, `fr`, `de`, `it`, `pt`, `ja`, `ko`, `zh`, `ar`, or `ru` |
| `searchQueries` | array | – | Advanced raw Maps queries; overrides `keyword` and `category` |
| `proxyConfiguration` | object | – | Optional Apify Proxy configuration for the run |

## Output per lead

One dataset record per business with only fields supported by observed data:

- Maps place data: `name`, `googleBusinessCategories`, `street`, `city`, `state`, `zip`, `country`, `countryCode`, `phoneNumber`, `url`, `reviewScore`, `reviewsNumber`, `placeId`, `featureId`, `googleMapsUrl`, `priceLevel`, `plusCode`, and `location`.
- Optional website enrichment (`extractEmails: true`): `email`, `emails`, `websitePhones`, `contactPageUrl`, `websitePagesCrawled`, `facebook`, `instagram`, `linkedin`, `twitter`, `youtube`, and `tiktok`.
- Record context: `recordType`, `sourceUrl`, and `scrapedAt`.

These are 32 possible keys; each is omitted when its source is unavailable. The
dataset table exposes the same field inventory.

A canonical Google Place ID beginning with `ChIJ` is recorded in `placeId`; a
legacy feature/CID token (`0x…:0x…`) is recorded in `featureId` and is never
relabelled as a Place ID. When both are observed, the canonical token takes
`placeId` and the legacy token is still recorded as `featureId`.

Website contact fields come from the optional public website crawl, not from
Google Maps itself. Unavailable fields are omitted rather than filled with
nulls or inferred values.

## Examples

### Example: London bakery enrichment

```json
{"searchQueries":["bakeries in London"],"max_results":50,"extractEmails":true}
```

### Example: Maps-only international search

```json
{"keyword":"dentist","country":"DE","extractEmails":false,"language":"de"}
```

### Example: Japanese prospecting without website enrichment

```json
{"searchQueries":["coffee shops in Tokyo","bakeries in Tokyo"],"max_results":25,"extractEmails":false,"language":"ja"}
```

## Use cases

- Build local prospecting lists with public business details.
- Enrich place results with contact information from each business website.
- Compare categories and locations across several search queries.

## Limitations

- Optional Google Maps fields are omitted when Google does not publish them, rather than filled with empty or placeholder values.

It is your responsibility to use this public data lawfully and in line with Google's terms.

## FAQ

**Q: Do I need a proxy?**
A: No. `proxyConfiguration` is available in the input if you want to route
traffic through your own Apify Proxy setup, but a default run works without it.

**Q: Why are some fields missing?**
A: We omit fields we couldn't populate rather than emit `null`. Every field
in your record has real data.

**Q: Does enrichment slow things down?**
A: Website crawl adds ~1-3s per business. Disable `extractEmails` for a
Maps-only run.

**Q: Is website contact data from Google Maps?**
A: No. Google Maps supplies the place record; email and social fields are copied
from publicly visible pages on the business website when enrichment is enabled.

**Q: Can I process several search queries together?**
A: Yes. Provide multiple `searchQueries`; matching businesses are deduplicated
by observed Google identity where available.
