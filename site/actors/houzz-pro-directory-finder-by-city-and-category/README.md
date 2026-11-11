# Houzz Professionals, Reviews & Photos Scraper

Scrape **Houzz** professional directories across 15 country sites, open each pro's profile for contact, service, award, social and licensing detail, pull their full review history, and browse the photo inspiration library by room, style or keyword. Everything is public and cookieless: no Houzz account, browser, API key or proxy setup is needed.

## What this actor does

- **Five modes:** `search`, `byUrls`, `profiles`, `reviews`, `photos`
- **Every live market:** US, Canada, UK, Ireland, Australia, New Zealand, Germany, France, Italy, Spain, India, Singapore, Japan, Denmark and Sweden, with localised profile paths
- **Search by city + profession or keyword**, with rating and review-count filters and server-side sort
- **Deep directory pagination:** up to 100 pages per category and 30 pages for keyword-filtered searches
- **Rich profiles:** business name, contact person, phone, website, full address, services, awards, badges, credentials, social links, licence status, project/photo/follower counts and review aspect scores
- **Full reviews:** reviewer, rating, body, project date and price, review photos and the professional's reply
- **Photo inspiration:** room category, keyword, 19 design styles, optional detail enrichment for full image sets, owner and location
- **Optional email enrichment** from each professional's own website
- Empty fields are omitted from every record instead of being returned as null

## Output per professional

- `title`, `businessName`, `description`
- `phoneNumber`, `websiteUrl`, `contactName`, `contactEmail`, `emails[]`, `emailSourceUrl`
- `licenseNumber`, `licenseType`, `licenseVerifiedDate`, `licenseIssuedBy`, `verifiedLicense`
- `ratingValue`, `reviewCount`, `hiresOnHouzz`, `followersCount`, `projectCount`, `photoCount`, `galleryCount`
- `isProVerified`, `isPaidPro`, `reviewAspectRatings{workQuality, communication, onTime, value}`
- `servicesProvided[]`, `awards[]`, `badges[]`, `credentials[]`, `faq[]`
- `socialLinks{facebook, instagram, twitter, pinterest, youtube, linkedin, tiktok}`
- `proId`, `userId`, `userName`, `category`, `categories[]`
- `streetAddress`, `city`, `state`, `postalCode`, `country`, `countryCode`, `fullAddress`
- `latitude`, `longitude`, `areaServed[]`, `imageUrl`
- `reviews[]`, `reviewsFetched`, `reviewsTotalAvailable` when reviews are requested
- `sourceUrl`, `profileUrl`, `canonicalUrl`, `slug`
- `siteName`, `recordType: "professional"`, `recordSource`, `scrapedAt`

## Output per review

- `reviewId`, `proId`, `proName`, `profileUrl`
- `rating`, `body`, `relationship`, `projectDate`, `projectPrice`
- `reviewerName`, `reviewerUserName`, `reviewerProfileUrl`, `reviewerIsPro`
- `createdAt`, `numberOfLikes`, `photos[]`
- `proResponse`, `proResponseAt`
- `recordType: "review"`, `scrapedAt`

## Output per photo

- `photoId`, `title`, `description`, `url`, `canonicalUrl`
- `imageUrl`, `heroImageUrl`, `images[]`, `imageWidth`, `imageHeight`
- `categoryId`, `category`, `styleId`, `style`
- `ownerId`, `ownerName`, `ownerProfileUrl`, `ownerIsPro`
- `projectId`, `questionCount`, `comments`, `recentBuzz`
- `metroArea`, `city`, `state`, `countryCode`
- `isFeatured`, `isGated`, `sourceUrl`
- `recordType: "photo"`, `scrapedAt`

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `search` | `search` / `byUrls` / `profiles` / `reviews` / `photos` |
| `country` | string | `us` | Houzz market: `us`, `ca`, `uk`, `ie`, `au`, `nz`, `de`, `fr`, `it`, `es`, `in`, `sg`, `jp`, `dk`, `se`; legacy `ru` is accepted for saved tasks but Houzz Russia is offline |
| `city` | string | `Los Angeles, CA` | City or city + state for search mode |
| `profession` | string | `general-contractor` | 16 universal professions mapped to each market |
| `categorySlug` | string | – | Exact Houzz category slug override |
| `keyword` | string | – | Professional keyword filter or photo search term |
| `sortBy` | string | `relevance` | `relevance` / `reviews` / `rating` / `distance` |
| `minRating` | number | `0` | Drop professionals below this rating |
| `minReviewCount` | integer | `0` | Drop professionals below this review count |
| `directoryUrls` | array | `[]` | Houzz directory, category or search-result URLs |
| `profileUrls` | array | `[]` | Direct Houzz professional profile URLs |
| `photoUrls` | array | `[]` | Direct Houzz photo detail URLs |
| `photoCategory` | string | `kitchen` | Room category for photo browsing |
| `photoStyle` | string | – | Optional design style filter |
| `includeReviews` | boolean | `false` | Attach review history to professional records |
| `maxReviewsPerPro` | integer | `20` | Review cap per professional; `0` means all |
| `fetchDetails` | boolean | `true` | Open each professional profile |
| `fetchPhotoDetails` | boolean | `false` | Open each photo for full image and owner detail |
| `enrichEmails` | boolean | `false` | Best-effort email lookup on the pro's own website |
| `maxItems` | integer | `25` | Hard record cap (1–5,000) |
| `maxPages` | integer | `0` | Optional page cap; `0` uses the source ceiling |

### Example: contractors in Los Angeles

```json
{
  "mode": "search",
  "country": "us",
  "city": "Los Angeles, CA",
  "profession": "general-contractor",
  "minRating": 4.5,
  "minReviewCount": 10,
  "maxItems": 100
}
```

### Example: London interior designers with reviews

```json
{
  "mode": "search",
  "country": "uk",
  "city": "London",
  "profession": "interior-designer",
  "includeReviews": true,
  "maxReviewsPerPro": 50,
  "maxItems": 40
}
```

### Example: specific profiles and directory URLs

```json
{
  "mode": "byUrls",
  "directoryUrls": [
    "https://www.houzz.de/professionals/architekten/berlin-16-de-probr0-bo~t_11973~r_2950159"
  ],
  "profileUrls": [
    "https://www.houzz.com/professionals/general-contractors/goldenline-remodeling-pfvwus-pf~28977151"
  ],
  "fetchDetails": true,
  "maxItems": 30
}
```

### Example: kitchen photo inspiration

```json
{
  "mode": "photos",
  "photoCategory": "kitchen",
  "photoStyle": "scandinavian",
  "fetchPhotoDetails": true,
  "maxItems": 60
}
```

## Use cases

- **Contractor lead generation** — build a city list with phone, website, address, licence and review proof
- **Competitive benchmarking** — compare services, awards and review scores across firms in one category
- **Market expansion research** — compare the same profession across 15 Houzz country sites
- **Vendor due diligence** — check credentials, licence status, project volume and owner replies
- **Review intelligence** — export reviewer sentiment, project budgets and response behaviour
- **Design trend research** — collect photo records by room, style and metro area for moodboards or analysis

## FAQ

**Is this affiliated with Houzz?**
No. This is an independent third-party actor that reads publicly visible Houzz pages. It is not operated by or affiliated with Houzz.

**Do I need a Houzz account, cookies or an API key?**
No. Every supported mode works anonymously on public pages. There is no login step.

**Do I need to configure a proxy?**
No. The actor makes direct requests and does not require proxy setup. Just choose a mode and run it.

**Why does Houzz Japan sometimes return no professionals?**
Houzz Japan intermittently blocks datacenter IPs or serves 404 profile pages. The actor reports the failed fetch and keeps any directory cards it already parsed instead of inventing rows. Other 14 markets run directly.

**Why do some records have fewer fields than others?**
Professionals publish different amounts of information, and some categories show fewer badges or services. Fields with no value on the source page are omitted, never returned as null.

**How many professionals can one search return?**
A category or city directory serves up to 100 pages, and a keyword-filtered search stops at 30 pages. Coverage also depends on how many professionals are actually active in that city and profession.

**Why is `phoneNumber` sometimes a toll-free number?**
Houzz may display a business tracking number on the profile while the directory card shows a local number. The actor prefers the local directory number when both are available.

**Why do photo keyword searches stop before a room browse does?**
Houzz caps photo keyword results at 200 items (10 pages), while room browsing goes much deeper. The actor reports the ceiling instead of silently returning fewer photos.

**Why is there no product mode?**
Shop Houzz closed in 2026. The product pages now show "No Longer Available" or return 404, so there is no live product data to scrape. Professional and photo data are unaffected.

**Can I combine several inputs in one run?**
Yes. Directory URLs and profile URLs can be mixed, and `search` accepts a city plus profession while also honouring any direct URLs you add.
