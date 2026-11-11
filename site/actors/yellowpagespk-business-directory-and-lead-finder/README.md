# YellowPagesPK Business Directory & Lead Finder

YellowPagesPK.com is one of Pakistan's largest online business directories. This actor turns it into structured lead data: browse a category, browse a city, run the site's keyword search, or open specific listing URLs. It returns the full public profile behind each business — contact details, addresses, category terms, opening hours, social profiles, photo galleries and map coordinates — with no login, cookies, API key or proxy setup.

## What it does

- **Four modes:** `byCategory` (browse one or more category archives), `byCity` (browse one or more city/location archives), `search` (the site's relevance-ranked keyword search) and `byUrls` (open specific listing pages).
- **All 47 published categories** and **all 65 published location terms**, exposed as multi-select dropdowns with human-readable titles.
- **The whole directory:** leaving `categories` or `cities` empty crawls every published listing through the site's WordPress REST API — currently **2,198 businesses**.
- **Combined targeting:** pick several categories and several cities in one run; city and category selections also act as exact filters in the other modes.
- **Lead-quality filters:** minimum rating, claimed-only, and only-with-email / only-with-website / only-with-phone.
- **Full profile records:** description, primary category plus category arrays, city/country, address, phone, email, website, WhatsApp and social profiles, weekday opening hours, open/closed state, rating and real review count, price range, photos, coordinates, author and publish dates.
- **Safe by design:** strict listing-URL validation, duplicate removal by WordPress post ID, and empty fields omitted rather than emitted as `null`.

## Services covered

Restaurants, hotels and guest houses, health and medical providers, education institutes, software houses, shopping, real estate, law firms, travel agencies, manufacturers, movers and packers, beauty and spa, automotive, electronics, construction, digital media and every other category the directory publishes.

## How it works

The actor uses two public, cookieless data paths:

1. **Server-rendered archives** for discovery — category archives and city archives return 100 business cards per page; the native search returns listing teasers 10 per page. Cards carry name, address, phone, coordinates, claimed/sponsored badges and images.
2. **WordPress REST API** for exact metadata — category and location term IDs, publish/modified dates, descriptions and totals. Term filters are exact, so a "Lahore + restaurant" request matches the directory's own taxonomy rather than a text guess. The REST endpoint has 2,198 published listings at 100 per request.

When `fetchDetails` is on, each business page is opened for the rich profile fields. Detail pages are the only source of email, website, opening hours, social links, gallery and price range. A sitemap index provides a fallback enumeration path if the REST endpoint is unavailable.

Requests use full browser headers because the site's LiteSpeed edge rejects incomplete requests with HTTP 406. Retries cover transient 5xx responses. Apify's datacenter proxy is used automatically only after a real block; no proxy configuration is needed from the user.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | select | `byCategory` | `byCategory`, `byCity`, `search` or `byUrls` |
| `categories` | multi-select | – | 47 category slugs; in `byCategory` restaurant when omitted, empty means the whole directory; exact filter in other modes |
| `cities` | multi-select | `[]` | 65 location slugs; browse in `byCity`, exact filter elsewhere |
| `searchQuery` | text | – | Keyword for `search` mode |
| `listingUrls` | URL list | `[]` | Exact YellowPagesPK listing URLs for `byUrls` |
| `minRating` | integer 0–5 | `0` | Minimum published rating; profiles are fetched automatically when the selected path cannot supply ratings |
| `onlyClaimed` | boolean | `false` | Keep only owner-claimed listings; profiles are fetched automatically when the selected path cannot supply claim badges |
| `onlyWithEmail` | boolean | `false` | Keep only records with an email (turns on profiles) |
| `onlyWithWebsite` | boolean | `false` | Keep only records with a website (turns on profiles) |
| `onlyWithPhone` | boolean | `false` | Keep only records with a phone (turns on profiles) |
| `fetchDetails` | boolean | `true` | Open each profile page for full contact/social/hours data |
| `maxItems` | integer 1–5,000 | `30` | Maximum records to return |

### Examples

Browse every restaurant in Lahore:

```json
{
  "mode": "byCategory",
  "categories": ["restaurant"],
  "cities": ["lahore"],
  "maxItems": 100
}
```

Browse software houses and digital media in Karachi and Islamabad:

```json
{
  "mode": "byCity",
  "cities": ["karachi", "islamabad"],
  "categories": ["software-houses", "digital-media"],
  "maxItems": 200
}
```

Search for a service, with contactable businesses only:

```json
{
  "mode": "search",
  "searchQuery": "digital marketing lahore",
  "onlyWithEmail": true,
  "maxItems": 50
}
```

Open specific profiles:

```json
{
  "mode": "byUrls",
  "listingUrls": [
    { "url": "https://yellowpagespk.com/listing/bizlaw/" },
    { "url": "https://yellowpagespk.com/listing/andaaz-restaurant/" }
  ]
}
```

Crawl the entire directory as a fast list of names, categories, cities, descriptions and dates (no profile requests):

```json
{
  "mode": "byCategory",
  "categories": [],
  "fetchDetails": false,
  "maxItems": 5000
}
```

## Output

Each dataset record is one business. Fields include:

`postId`, `businessId`, `name`, `description`, `category`, `categorySlug`, `categories`, `categorySlugs`, `searchCategory`, `searchQuery`, `address`, `city`, `country`, `locationTerms`, `phone`, `email`, `website`, `latitude`, `longitude`, `rating`, `reviewCount`, `openStatus`, `openNow`, `openingHours`, `socialLinks`, `images`, `imageUrl`, `logoUrl`, `priceRange`, `claimed`, `sponsored`, `authorName`, `authorProfileUrl`, `publishedAt`, `lastUpdated`, `sourceUrl`, `recordType`, `scrapedAt`.

All nested values are omit-empty: a business without a website has no `website` key, and a profile without a gallery has no `images` key.

## Usage cost

The actor makes cheap HTML/JSON requests only; it does not launch a browser and does not use paid proxy groups. A normal category run with profiles enabled costs one archive request plus one request per business. `onlyWith*` filters, and rating/claimed filters in modes whose fast path lacks those fields, enable profiles automatically. For the lowest-cost large runs, use category/city archives with `fetchDetails` set to `false` and `maxItems` up to 5,000. Apify compute and the actor's own pay-per-result pricing (when enabled on the listing) are billed by the platform according to your plan.

## FAQ

**Do I need an account, cookies or an API key?**
No. Every page and endpoint used by this actor is public.

**How many businesses can I get?**
The directory currently has 2,198 published listings. Leave `categories` or `cities` empty to crawl all of them; otherwise the selected archives are fully paginated.

**Why do some records have no email, website, hours or rating?**
The directory only shows what each business supplied. Email and website appear on profile pages; hours are published by a small minority; ratings exist only after a review. Missing values are omitted, never invented.

**What is the difference between `byCategory` and `byCity`?**
They are two browse paths over the same listings. `byCategory` walks category archives and can be narrowed with `cities`; `byCity` walks location archives and can be narrowed with `categories`. Both deduplicate by post ID.

**Does `fetchDetails` change the output?**
Yes. With it off, category/city archive runs still return card + REST fields: name, address, phone, categories, city/country, coordinates, badges and dates. Keyword-search runs return name, image, categories, city/country, description and dates; whole-directory REST runs return name, categories, city/country, description and dates. With it on, every profile is opened for email, website, hours, socials, gallery and price range. Contact filters and filters whose field is missing from the selected fast path turn profiles on automatically.

**How is city chosen when the site lists a state or country?**
The actor prefers a real city among the listing's location terms, then matches the address against a curated city list. Country is derived separately, so a Seattle business tagged "Washington" is returned as `city: "Seattle"` and `country: "United States"`, not as a city named Washington.

**Are duplicate businesses possible across categories or cities?**
No. Records are deduplicated by WordPress post ID (and canonical URL) within a run.

**What happens if a detail page fails?**
The card/REST fields are kept and the run continues; the log reports how many profile requests failed.

## Notes

- `businessId` is always the listing slug and `postId` is always the numeric WordPress ID, in every mode.
- `searchCategory` records the category you browsed; `searchQuery` records the keyword you searched. `categories`/`categorySlugs` always describe the business itself.
- `reviewCount` is parsed from the profile's "N Ratings" label and is only present when the site publishes it.
- The default input is deliberately small (two restaurants, profiles off) so scheduled runs stay cheap; raise `maxItems` and enable `fetchDetails` for production jobs.

## Limitations

- **Address coverage is partial.** Roughly half of all listings publish a street address; the rest only have a city/country term. Address is omitted when the source has none.
- **Ratings are rare.** Only a handful of listings have a rating and review count. Review text is not published anywhere on the site, so it cannot be scraped.
- **Opening hours are rare.** Most profiles do not publish weekday hours; when they do, only the days shown are returned (no day is invented).
- **City and country are best-effort.** The location taxonomy mixes cities, provinces, countries and a few neighbourhoods. The actor picks the most specific city term and derives country separately; listings with no location data may have no city.
- **Some images are broken at the source.** Older photos and logos referenced by the site may return 404. Gallery and logo URLs are checked before output when profiles are fetched, so a missing image reflects the source, not a scrape failure; card thumbnails in fast runs are passed through as published.
- **2,198 listings is the current directory total**, not a cap imposed by the actor. The REST enumeration includes the two listings that do not appear in category archives; if REST fails, a sitemap fallback enumerates listing URLs.
- **No sort or resume options.** The site offers no reliable server-side sort or cursor, so results follow the directory's own archive order.
