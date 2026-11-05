# Google Maps Radius Search Around a Coordinate

Give this actor a latitude/longitude and a category, and it returns the places Google Maps shows within a chosen radius of that exact point — each tagged with its straight-line distance from the center. It's built for "what's within N km of this spot" lookups rather than broad city-wide searches.

## What this actor does

- Searches for a category or free-text query (e.g. `coffee shop`, `pharmacy`, `gas station`) centered on a latitude/longitude you provide
- Enforces the search radius as a hard filter — any place Google returns outside the radius is dropped, and every kept place gets a `distanceFromCenter` and `distanceFromCenterMeters` value
- Filters by minimum rating and by whether Google currently shows the place as open
- Picks a Google Maps zoom level based on your radius, so a 0.3 km radius returns tighter local results than a 5 km one
- Omits any field Google doesn't show for a place rather than sending it empty

## Output per place

- `name`
- `categoryName` — the place's category as labeled on Google Maps
- `totalScore` — average rating
- `reviewsCount`
- `address`
- `phone`
- `website`
- `priceLevel` — e.g. `$$` or a price range, when Google shows one
- `isOpenNow` — `true`/`false` when Google's open/closed status could be read
- `placeId` — canonical Google Place ID (`ChIJ…`), when available
- `featureId` — legacy Google feature identifier, used when no canonical Place ID is exposed
- `plusCode`
- `location` — `{ lat, lng }` of the place itself
- `distanceFromCenter` — human-readable distance from your input coordinate, e.g. `450 m`
- `distanceFromCenterMeters` — the same distance as a number
- `googleMapsUrl`
- `recordType` — always `business`
- `sourceUrl`
- `scrapedAt` — ISO 8601 timestamp of when the record was collected

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `category` | string | – (required) | Type of place to search for near the given location |
| `location` | object | – (required) | Center point as `{ "lat": ..., "lng": ... }` |
| `radiusKm` | number | `1` | Search radius in kilometers (0.1-10); places outside it are dropped |
| `maxResults` | integer | `30` | Maximum number of places to return (1-120, Google Maps' own per-query ceiling) |
| `minRating` | number | – | Only include places rated at or above this value (1.0-5.0) |
| `openNow` | boolean | `false` | Only include places Google currently shows as open; places whose status can't be read are dropped too |
| `language` | string | `en` | Interface/results language: `en`, `es`, `fr`, `de`, `it`, `pt`, `ja`, `ko`, `zh`, `ar`, or `ru` |
| `proxyConfiguration` | object | – | Optional proxy; datacenter IPs usually work fine for Google Maps |

### Example: tight radius around a point

```json
{
    "category": "pharmacy",
    "location": { "lat": 40.7484, "lng": -73.9857 },
    "radiusKm": 0.5,
    "maxResults": 20
}
```

### Example: open-now, highly rated restaurants nearby

```json
{
    "category": "restaurant",
    "location": { "lat": 51.5074, "lng": -0.1278 },
    "radiusKm": 2,
    "minRating": 4.5,
    "openNow": true,
    "maxResults": 30
}
```

### Example: wider sweep in another language

```json
{
    "category": "供应商",
    "location": { "lat": 35.6762, "lng": 139.6503 },
    "radiusKm": 5,
    "language": "ja",
    "maxResults": 60
}
```

## Use cases

- **Site-selection scans** — check what's already within walking distance of a candidate storefront location
- **Delivery radius audits** — list every matching business inside a courier's actual delivery radius, not an entire city
- **Point-of-interest enrichment** — attach the nearest matching businesses to a dataset of coordinates (stores, offices, listings)
- **Open-now availability checks** — find which nearby places of a category are actually open right now
- **Competitive density mapping** — count and rate competitors within a fixed distance of a specific address

## FAQ

**How is this different from a regular Google Maps text search?** A text search ranks by relevance across a wide area; this actor is anchored to one exact coordinate and radius, drops anything Google returns outside that radius, and reports each place's distance from your center point.

**Why did I get fewer results than `maxResults`?** Google Maps caps a single query at 120 results, and this actor additionally removes anything outside your `radiusKm`, below `minRating`, or (with `openNow` set) not confirmed open — so a tight radius or strict filters can legitimately return fewer rows, including zero.

**Why is `isOpenNow` sometimes missing even without `openNow` filtering?** It's only included when Google's page actually shows a readable open/closed indicator for that place; some listings don't expose one.

**What's the difference between `placeId` and `featureId`?** `placeId` is Google's canonical `ChIJ…` identifier. `featureId` is a legacy identifier Google sometimes returns instead; a record carries whichever one Google actually exposed for that place.

**Does a smaller radius give more accurate local results?** Yes — the radius also sets the map zoom level used for the search, so a smaller radius naturally surfaces tighter, more local matches rather than ones from the edge of a wide area.

**Do I need a proxy?** No. Datacenter IPs work for most runs; the proxy input exists only for cases where your requests start getting blocked.

**Is this affiliated with Google?** No. This actor reads Google Maps' publicly visible listing pages and is not affiliated with, endorsed by, or sponsored by Google.
