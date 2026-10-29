# Google Maps Related Place & Competitor Finder

When you open a business, hotel, or landmark on Google Maps, Google often shows a "People also search for" or similar-places carousel next to it. This actor reads that carousel for any place you point it at and returns each related place as its own record, linked back to the place you searched.

## What this actor does

- Looks up a Google Maps place by URL or canonical Place ID and reads its related-places carousel.
- Returns each related place's name, identifiers, rating, review count, category, and coordinates.
- Keeps the carousel's own display order as a `rank` field.
- Links every related place back to the source business so a multi-run dataset stays joinable.
- Supports 11 Maps interface languages, parsing localized ratings, review counts, price, and distance without translating the values.
- Omits a field whenever Google doesn't publish it for that place, rather than sending it empty or guessed.

## Output per related place

- `rank` — position in the carousel (1 = first card shown)
- `relatedSectionType` — the carousel's own label, e.g. "People also search for" or "Similar places"
- `name`
- `placeId` — canonical Google Place ID (`ChIJ…`), present only when Google publishes one
- `featureId` — legacy Google feature identifier (`0x…:0x…`), present only when Google exposes it; never used in place of `placeId`
- `googleMapsUrl` — link to the related place, from the card itself or built from its identifiers when needed
- `categoryName`, `categories` — published category and any additional category list
- `totalScore`, `reviewsCount` — rating and review count
- `price` — price indicator, when shown
- `description` — short description text, when shown
- `distance` — distance from the source place, when shown
- `latitude`, `longitude`
- `image` — thumbnail image URL, requested at a larger resolution when the source URL supports it
- `sourceBusinessName`, `sourcePlaceId`, `sourceFeatureId`, `sourcePlaceUrl` — identity of the place you searched, repeated on every related-place record
- `sourceUrl` — the source place's URL
- `recordType` — always `similarPlace`
- `scrapedAt`

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `placeUrl` | string | – | A Google Maps place URL; use this or `placeId` |
| `placeId` | string | – | Canonical Google Place ID beginning with `ChIJ`; use instead of `placeUrl` |
| `maxResults` | integer | `20` | Maximum number of related places to return (1-50) |
| `language` | string | `en` | Maps interface language: `en`, `es`, `fr`, `de`, `it`, `pt`, `ja`, `ko`, `zh`, `ar`, or `ru` |
| `proxyConfiguration` | object | `{"useApifyProxy": true}` | Optional Apify Proxy configuration for the run |

### Example: related hotels from a place URL

```json
{
  "placeUrl": "https://www.google.com/maps/place/The+Ritz+London/@51.5070782,-0.1416273,17z/data=!3m1!4b1!4m6!3m5!1s0x48760529d30fc857:0x596135811e044014!8m2!3d51.5070782!4d-0.1416273!16zL20vMDQ3enJ2?hl=en",
  "maxResults": 10,
  "language": "en"
}
```

### Example: related places by canonical Place ID

```json
{
  "placeId": "ChIJabtoT7hlwokRLECK0SW3_8I",
  "maxResults": 20
}
```

### Example: related places in a non-English market

```json
{
  "placeUrl": "https://www.google.com/maps/place/Sushi+no+Midori/@35.6704396,139.7596577,17z/",
  "maxResults": 10,
  "language": "ja"
}
```

## Use cases

- Competitive analysis — see which hotels, restaurants, or stores Google associates with a given business.
- Build a "places like this" recommendation feed from Google's own related-place signal.
- Turn one landmark into a shortlist of nearby related landmarks or attractions.
- Expand a small seed list of places into a wider set for further research.
- Local SEO — check which competitors show up alongside your own listing.

## FAQ

**Does this need a login, cookies, or an API key?**
No. It reads Google Maps' public place pages.

**Does this need a proxy?**
No. The default run works without one; `proxyConfiguration` is available in the input if you want to supply your own.

**Will every place have related places?**
Not always — whether Google shows this carousel at all, and how many cards it has, varies by place and by run. If none are available, the run finishes with zero related-place records rather than inventing any.

**How many related places can I get per place?**
Google typically shows a handful of cards per place; `maxResults` (up to 50) caps how many of those this actor returns.

**Why is `placeId` missing on some related places but `featureId` present?**
Google doesn't publish a canonical Place ID for every place shown in a related-places carousel. When only the legacy feature identifier is available, the record carries `featureId` instead, and `placeId` is left out rather than guessed.

**Does this work outside English-language Maps?**
Yes — set `language` to any of the 11 supported codes. Ratings, review counts, prices, and distances are read from that locale's own text and numerals rather than being translated.

**Can I process many places in one run?**
Each run takes one place. To cover several, run the actor once per place (for example from a script or scheduler); every run's related-place records carry `sourcePlaceId`/`sourceFeatureId` so you can join the results afterward.

**Is this affiliated with Google?**
No. This actor reads Google Maps' public pages and is not operated by or connected to Google.
