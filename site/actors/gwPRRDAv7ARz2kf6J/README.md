# Google Workspace Marketplace Category Tracker

Track ranked apps across every Google Workspace Marketplace category, run keyword searches, look up specific apps, and pull their public reviews — without a Google account, API key or proxy.

The actor reads the same public pages and embedded JSON the marketplace serves to a browser. Listings are capped at 100 apps per request, so the actor automatically unions Google's own rating, pricing and "works with" slices to reach deeper into a category when you ask for more rows.

## What it does

- **Category tracking** — 27 categories and curated lists (Most Popular, Top Rated, Featured Partner Apps, Business Tools, Productivity, Education, Connectors and every subcategory), with site rank preserved for the default slice.
- **Keyword search** — one or more queries, ranked separately and deduplicated.
- **App lookup** — pass numeric IDs or full marketplace URLs and get the complete public listing.
- **Reviews** — paginated review rows (10 at a time) with rating, text, date, helpful votes and reviewer name; filter by star range and date, sort by helpful or newest.
- **Real filters** — works-with host, pricing model, minimum rating, review count, installs and developer substring.
- **Extra detail** — exact OAuth scopes, sensitive-access flags and the independent security-verification badge for each app.
- **Deep listings** — when `maxItems` exceeds what the default page returns, the actor fetches additional rating/pricing/host slices and deduplicates by app ID.
- **Omit-empty output** — a field the marketplace does not publish is left out of that record, never `null` or a placeholder.

## Modes

| Mode | What it returns | Required input |
|---|---|---|
| `category` (default) | Ranked apps from one or more categories or curated lists | `categories` (defaults to `popular-apps`) |
| `search` | Ranked apps for one or more keyword queries | `searchQueries` |
| `appDetails` | Full records for specific apps | `appIds` |
| `reviews` | Review rows only | `appIds` |

Add `includeReviews` to `category`, `search` or `appDetails` to emit app rows and review rows in the same run.

## Input options

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | select | `category` | What to scrape |
| `categories` | multi-select | `["popular-apps"]` | Category/curated-list slugs (mode=category) |
| `searchQueries` | string list | — | Keywords (mode=search) |
| `appIds` | string list | — | Numeric IDs or marketplace URLs (appDetails/reviews) |
| `worksWith` | select | `any` | Drive, Docs, Sheets, Slides, Forms, Gmail, Calendar, Chat, Meet, Classroom, Gemini, Studio |
| `pricing` | select | `any` | Free, Free trial, Freemium, Paid |
| `minRating` | number | `0` | Keep apps rated at least this (0–5) |
| `minReviews` | integer | `0` | Keep apps with at least this many ratings |
| `minInstalls` | integer | `0` | Keep apps with at least this many exact installs |
| `developerFilter` | text | — | Keep apps whose developer name or website contains this text |
| `includeDetails` | boolean | `false` | Add OAuth scopes, sensitive-access flags and security badge |
| `includeReviews` | boolean | `false` | Emit review rows for matched apps |
| `sortBy` | select | `helpful` | `helpful` or `recent` |
| `minReviewRating` | integer | `1` | Lowest review star rating to keep |
| `maxReviewRating` | integer | `5` | Highest review star rating to keep |
| `reviewsSince` | text | — | ISO date, e.g. `2026-01-01` |
| `maxReviewsPerApp` | integer | `50` | Review cap per app |
| `includeCategoryMembership` | boolean | `false` | Scan all 27 listings to fill each app's `categories` |
| `maxItems` | integer | `100` | Max app rows (max review rows in `reviews` mode) |

## Output fields

### App rows (`recordType: "workspace_app"`)

| Field | Description |
|---|---|
| `rankInCategory` | 1-based rank in the category's default listing (omitted for apps only found in later slices) |
| `rankInResults` | 1-based rank in the keyword search default listing |
| `category` / `categories` | Selected category and (when resolved) the app's category memberships |
| `searchQuery` / `searchQueries` | Keyword that matched the app |
| `appId`, `slug`, `name` | Identity |
| `shortDescription`, `longDescription` | Listing descriptions |
| `developer`, `developerWebsite` | Publisher |
| `rating`, `reviewCount` | Average stars and rating count |
| `installCount`, `installsLabel` | Exact installs (e.g. `97456341`) and the public band (e.g. `97M+`) |
| `pricingModel` | `Free`, `Free trial`, `Freemium` or `Paid` (omitted when the listing does not state pricing) |
| `worksWith` | Google apps the listing works with |
| `lastUpdated` | Listing update date (`YYYY-MM-DD`) |
| `iconUrl`, `bannerUrl`, `screenshots` | Public image URLs |
| `supportUrl`, `privacyPolicyUrl`, `termsOfServiceUrl` | Links published on the listing |
| `appUrl`, `sourceUrl` | Canonical marketplace app page |
| `securityVerified` | `true` when the independent security-verification badge is shown |
| `oauthScopes`, `oauthScopeCount`, `sensitiveAccess` | Requested OAuth scopes and flags such as `gmailFullAccess` |
| `recordType`, `scrapedAt` | Row metadata |

### Review rows (`recordType: "workspace_review"`)

`reviewId`, `appId`, `appName`, `appUrl`, `reviewerName`, `reviewerAvatarUrl`, `rating`, `reviewText`, `reviewDate`, `helpfulCount`, `sourceUrl`, `scrapedAt`.

## Example inputs

Track the top 50 apps in a category, including OAuth permissions:

```json
{
  "mode": "category",
  "categories": ["business-tools/sales-and-crm"],
  "maxItems": 50,
  "includeDetails": true
}
```

Deep-scan productivity with a filter, plus reviews:

```json
{
  "mode": "category",
  "categories": ["productivity"],
  "worksWith": "sheets",
  "minRating": 4,
  "maxItems": 250,
  "includeReviews": true,
  "sortBy": "recent",
  "maxReviewRating": 2,
  "maxReviewsPerApp": 20
}
```

Look up apps by URL or ID:

```json
{
  "mode": "appDetails",
  "appIds": [
    "https://workspace.google.com/marketplace/app/lucidchart/7081045131",
    "1014160490159"
  ],
  "includeDetails": true
}
```

Pull 100 newest reviews for one app:

```json
{
  "mode": "reviews",
  "appIds": ["7081045131"],
  "sortBy": "recent",
  "maxItems": 100
}
```

## Example output

```json
{
  "recordType": "workspace_app",
  "rankInCategory": 1,
  "category": "productivity",
  "categories": ["productivity"],
  "appId": "1014160490159",
  "slug": "colaboratory",
  "name": "Colaboratory",
  "shortDescription": "This allows Google Colaboratory to open and create files in Google Drive.",
  "developer": "Colaboratory team",
  "developerWebsite": "https://colab.research.google.com",
  "rating": 4.68,
  "reviewCount": 3824,
  "installCount": 97456341,
  "installsLabel": "97M+",
  "worksWith": ["Drive"],
  "lastUpdated": "2025-08-25",
  "iconUrl": "https://lh3.googleusercontent.com/bY20qlnjiFpsT_MtcuOGIAZ70lsnGe00yNVFF4hWT3QCz0-r7Rk",
  "appUrl": "https://workspace.google.com/marketplace/app/colaboratory/1014160490159",
  "sourceUrl": "https://workspace.google.com/marketplace/app/colaboratory/1014160490159",
  "scrapedAt": "2026-11-08T10:00:00+00:00"
}
```

## FAQ

**Do I need a Google account, API key or proxy?**
No. Every field comes from public marketplace pages and their embedded JSON. The actor makes direct HTTP requests and needs no credentials.

**How do I get more than 100 apps from one category?**
The marketplace returns at most 100 apps per request. The actor works around this by fetching the category through Google's own rating, pricing and works-with slices and merging the results. Set `maxItems` above 100 to enable the extra slices. Very broad categories can still be deeper than the slice union; split them by subcategory or add a works-with filter for full coverage of a narrower slice.

**Why is `rankInCategory` missing on some rows?**
Rank comes from the category's default (unsliced) page. Apps that only appear in a later slice have no position on that page, so the field is omitted rather than invented.

**Why do some apps have no `pricingModel`?**
Some listings do not state pricing. That row simply omits the field instead of guessing "Free".

**How many reviews can I get?**
The actor pages through the marketplace's review endpoint 10 at a time, up to `maxReviewsPerApp` (maximum 2,000). The marketplace may report slightly different totals between calls, so reviews are deduplicated by review ID and the actor stops when a page adds nothing new.

**Can I get developer replies to reviews?**
No. The public review payload does not contain developer replies, so the actor does not invent them.

**What does `includeCategoryMembership` do?**
The marketplace does not expose an app's category list on its app page. That option scans all 27 category listings and records which ones include the app. Because each listing shows at most 100 apps, membership is limited to those listing pages.

**What are `sensitiveAccess` flags?**
They summarise OAuth scopes: `gmailFullAccess`, `driveFullAccess`, `calendarFullAccess`, `contacts` and `adminDirectory`. Use them to audit what an add-on can access before installing it.

**Can I monitor changes between runs?**
Export the dataset on a schedule and compare snapshots. This actor returns current data; it does not store state.

## Limitations

- Each marketplace listing request is capped at 100 apps. Deep listing coverage is best-effort through Google's rating, pricing and works-with slices.
- Google ignores the Gemini and Studio host filters on listing URLs; those works-with values are applied to the rows the actor already fetched.
- Category membership is limited to the top-100 page of each category.
- Review totals reported by the marketplace fluctuate; review pagination stops at the reported total or when three consecutive pages add nothing new.
- Developer replies, emails, phone numbers and trader addresses are not published in the listing payload and are not fabricated.

## Data source

Public pages under `workspace.google.com/marketplace`: category and curated-list pages, search results, app detail pages, and the marketplace's own review endpoint. This actor is independent and not affiliated with Google.
