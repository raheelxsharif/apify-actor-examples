# Google Maps Top List & Ranked Places Extractor

Google Maps shows curated "Top" lists for queries like *Top hotels in Paris* or *Best coffee in Brooklyn*, each place ranked in the order Google displays it. This actor captures that ranked feed — from a text query or from a Google Maps list/search URL you already have — and can enrich every entry with the detail a place's own page publishes.

## What this actor does

- Runs a curated "top list" style query and returns the ranked feed Google shows for it.
- Scrapes a specific Google Maps list or search URL directly instead of a query.
- Keeps each place's `rank` so you can preserve Google's own ordering.
- Optionally visits each place's own page to add phone, website, exact address, coordinates, hours, amenities and more.
- Omits any field Google didn't show for that place — no record contains `null` or empty-string placeholders.

## Output per place

```json
{
  "recordType": "business",
  "listSource": "Top hotels in Paris",
  "rank": 1,
  "name": "Hôtel Le Bristol Paris",
  "categoryName": "5-star hotel",
  "totalScore": 4.7,
  "reviewsCount": 2891,
  "address": "112 Rue du Faubourg Saint-Honoré, 75008 Paris, France",
  "phone": "+33 1 53 43 43 00",
  "website": "https://www.oetkercollection.com/...",
  "priceLevel": "$$$$",
  "price": "PKR 980K",
  "placeId": "ChIJ...",
  "featureId": "0x...:0x...",
  "location": { "lat": 48.8718, "lng": 2.3163 },
  "googleMapsUrl": "https://www.google.com/maps/place/...",
  "highlights": ["Pool", "Spa", "Restaurant"],
  "amenities": ["Free Wi-Fi", "Paid breakfast", "Parking", "Accessible", "Indoor pool"],
  "sourceUrl": "https://www.google.com/maps/place/...",
  "scrapedAt": "2026-06-16T08:00:00+00:00"
}
```

Key fields explained:

- `placeId` is the canonical `ChIJ...` token. A legacy `0x...:0x...` token, when that's all the page exposes, is reported separately as `featureId` and never relabeled as `placeId`.
- `priceLevel` and `price` are different signals: `priceLevel` is Google's standing price band (`$$$$`), while `price` is whatever price string Google prints on the list card itself (often a nightly rate), returned verbatim since it's viewer-localized and can vary between page loads. It's omitted when no price is shown on the card.
- `highlights` is the short set of feature pills on the **list card** (typically 3–5 items). `amenities`, only present with `enrichDetails: true`, is the fuller set of amenity badges from the place's own page, including ones behind "View more amenities" — published exactly as Google displays them, with negative badges (e.g. "Pool unavailable") excluded.
- With `enrichDetails: true`, a record may also carry `description`, `plusCode`, `ratingHistogram`, `hours`, `hoursStatus`, `isOpenNow`, `serviceOptions`, `menuLink`, `bookingLinks`, `photosSample`, `photoCount`, `popularTimesSummary`, `claimed`, `temporarilyClosed`, and `permanentlyClosed`, whenever Google publishes them for that place.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | String (select) | `topList` | `topList` (text query) or `listUrl` (direct URL). |
| `query` | String | – | Curated list query, e.g. "Best Italian restaurants in Rome" (required when `mode=topList`). |
| `listUrl` | String | – | A Google Maps list or search URL (required when `mode=listUrl`). |
| `maxResults` | Integer | `30` | Requested ceiling on places returned (1–100,000); Google's own feed may expose fewer. |
| `enrichDetails` | Boolean | `true` | Visit each place's own page for full details. |
| `language` | String (select) | `en` | Google Maps interface language — `en`, `es`, `fr`, `de`, `it`, `pt`, `ja`, `ko`, `zh`, `ar`, or `ru`. |
| `proxyConfiguration` | Object | – | Optional proxy settings; the actor works without one in normal use. |

## Examples

### Curated hotel list

```json
{
  "mode": "topList",
  "query": "Top hotels in Paris",
  "maxResults": 10,
  "language": "en"
}
```

### Lightweight restaurant list (card data only)

```json
{
  "mode": "topList",
  "query": "Best sushi restaurants in Tokyo",
  "maxResults": 25,
  "enrichDetails": false,
  "language": "ja"
}
```

### A specific Google Maps list URL

```json
{
  "mode": "listUrl",
  "listUrl": "https://www.google.com/maps/search/top+coffee+shops+in+Seattle",
  "maxResults": 50
}
```

## Use cases

- Travel and hospitality research — compile top hotels or restaurants per city.
- Local SEO and competitor analysis for a category and area.
- Lead generation for hospitality or food-and-beverage outreach.
- Building "things to do" content from curated city lists.
- Site-selection research by comparing ranked places across cities.

## FAQ

**Does it need cookies or an API key?**
No, every mode works against public Google Maps pages.

**Will it work on the Apify free plan?**
Yes, no login or cookies are required.

**How many places can I actually get?**
You can set `maxResults` up to 100,000, but a single public Google feed commonly exposes around 120 or fewer cards; the actor keeps going if Google's feed exposes more.

**Will `highlights` show up for every place?**
No, Google only prints feature pills on certain card types, mainly hotels. Restaurants, cafés and attractions usually get a short description instead.

**How complete is `amenities`?**
It's the whole set of badges Google publishes on the place's own page with `enrichDetails: true`, including ones only revealed behind "View more amenities" — never a hand-picked subset. It's omitted entirely when Google shows no amenity strip for that place.

**Does it support non-English Google Maps?**
Yes, pick any of the 11 languages in the `language` dropdown.

**What happens if Google blocks the request or serves a degraded page?**
The run's status message tells the two cases apart: a genuinely empty result set reads "No places found for this query or list URL," while a page that never rendered a results view reads as an upstream block, so a failed fetch is never reported as a legitimate zero-result run.

**Is this affiliated with Google?**
No. This actor is an independent tool that reads publicly available Google Maps pages and is not affiliated with, endorsed by, or sponsored by Google.

You are responsible for using this public data lawfully and in keeping with Google's own terms.
