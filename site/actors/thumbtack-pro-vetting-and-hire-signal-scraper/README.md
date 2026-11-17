# Thumbtack Pro Vetting & Hire Signal Scraper

Build a shortlist of Thumbtack service pros from public search pages and profile pages. Every record carries reputation and hiring signals: star rating, review volume, hires booked through Thumbtack, response speed, Top Pro status, background-check status, years in business and services offered.

## What it does

- **Search by category and location** — one city, or several `City, ST` locations supplied in the same run.
- **Browse a category nationwide** — sweeps major US cities and deduplicates pros seen in more than one market.
- **Fetch specific profile URLs** — paste Thumbtack profile links for deep vetting records.
- **Fetch category page URLs** — paste front-door links such as `https://www.thumbtack.com/tx/austin/plumbers`.
- **Paginate and filter** — four server-side sort orders, optional continuation across sort orders and nearby cities, and filters for rating, reviews, hires, Top Pro, instant booking and background-check status.

## Output fields

| Field | Description |
| --- | --- |
| `proId` | Thumbtack service identifier. |
| `name` / `businessName` | Display and business name. |
| `serviceCategory` | Category the pro was found under. |
| `serviceDescription` | Profile introduction, when published. |
| `rating` / `reviewCount` | Average star rating and number of reviews. |
| `reviewQualifier` | Thumbtack's qualitative rating label. |
| `hireCount` | Number of hires booked through Thumbtack. |
| `responseTime` / `responseTimeHours` | Display text and numeric average response time. |
| `topPro` / `topProYears` | Top Pro status and the years awarded. |
| `isBackgroundChecked` | Whether the public profile shows a passed background check. |
| `isVerified` | Whether the profile shows a business verification credential. |
| `yearsInBusiness` / `numEmployees` | Business age and employee count. |
| `instantBookable` / `availabilityText` | Instant-booking signal from the card. |
| `priceText` / `startingPrice` / `priceUnit` | Published price, when the pro shows one. |
| `urgencySignals` | Thumbtack demand/value signals. |
| `services` / `specialties` | Flat service list and grouped specialties. |
| `credentials` | Background check and business verification entries. |
| `businessHours` / `paymentMethods` / `socialLinks` | Profile details, when published. |
| `ratingHistogram` / `reviews` | Star distribution and up to five highlighted reviews. |
| `mediaImages` / `numPhotos` | Sample project photos and total count. |
| `reviewText` / `reviewerName` | Featured review excerpt and author. |
| `city` / `state` / `serviceArea` | Location context. |
| `profileUrl` / `avatarUrl` | Public profile and image links. |
| `businessFacts` | Raw trust and activity facts shown by Thumbtack. |
| `scrapedAt` | ISO timestamp. |

Fields a pro does not publish are omitted, never filled with placeholders.

## Input

| Field | Type | Default | Description |
| --- | --- | --- | --- |
| `mode` | Select | `search` | `search`, `byCategory`, `byProfile` or `byUrl`. |
| `category` | Select | `house-cleaning` | Service category for search and byCategory modes. |
| `state` | Select | `CA` | Two-letter state code for the primary location. |
| `city` | String | `san-francisco` | City slug or `City, ST`. |
| `locations` | Array | `[]` | Extra `City, ST` locations for pagination and market coverage. |
| `profileUrls` | Array | `[]` | Profile URLs for byProfile mode. |
| `startUrls` | Array | `[]` | Category URLs for byUrl mode. |
| `sortBy` | Select | `recommended` | `recommended`, `highest_rated`, `most_hires` or `fastest_response`. |
| `minRating` | Number | — | Minimum average rating. |
| `minReviews` | Integer | — | Minimum review count. |
| `minHires` | Integer | — | Minimum Thumbtack hire count. |
| `topProOnly` | Boolean | `false` | Keep Top Pro badge holders only. |
| `backgroundCheckedOnly` | Boolean | `false` | Keep background-checked pros only; profiles are opened for candidates. |
| `instantBookOnly` | Boolean | `false` | Keep instant-bookable pros only. |
| `includeProfileDetails` | Boolean | `false` | Open every matched profile for deep vetting fields. |
| `paginate` | Boolean | `true` | Continue through sort orders and nearby cities until `maxItems`. |
| `maxItems` | Integer | `5` | Hard cap on records. |

### Example: top plumbers in New York with deep details

```json
{
  "mode": "search",
  "category": "plumber",
  "state": "NY",
  "city": "new-york",
  "sortBy": "most_hires",
  "minRating": 4.5,
  "minHires": 10,
  "includeProfileDetails": true,
  "maxItems": 20
}
```

### Example: background-checked cleaners across three cities

```json
{
  "mode": "search",
  "category": "house-cleaning",
  "locations": ["Austin, TX", "Dallas, TX", "Houston, TX"],
  "backgroundCheckedOnly": true,
  "maxItems": 30
}
```

### Example: nationwide roofers

```json
{
  "mode": "byCategory",
  "category": "roofing",
  "sortBy": "highest_rated",
  "maxItems": 50
}
```

## Pagination and location coverage

Thumbtack's public category page previews roughly 4-10 pros per city. With `paginate` enabled, the actor continues through the other three sort orders, then through nearby major cities, deduplicating on `proId`. Supply `locations` for deterministic coverage; use `byCategory` for a nationwide sweep.

## Notes and limits

- Every value comes from public Thumbtack pages, with no login, cookies or API key.
- Contact details (phone, email, street address) are not published on these pages and are not scraped.
- Deep fields such as background check and years in business require `includeProfileDetails` or a byProfile run and appear only when the pro publishes them.
- Reviews are capped at the five highlighted reviews Thumbtack shows publicly.
- A single city/category page is a preview, not the full directory; use locations, sort orders or byCategory to widen coverage.

## Data source

Thumbtack's public front-door category pages and public pro profile pages.
