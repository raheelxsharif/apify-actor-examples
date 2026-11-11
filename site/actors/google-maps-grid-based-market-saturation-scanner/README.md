# Google Maps Grid-Based Market Saturation Scanner

A single Google Maps search typically stops at around 120 listings no matter how large the area. This actor covers your target area with a grid of small search zones, reads each zone's full ranked result window, and deduplicates everything by Place ID — so a city-wide or region-wide query returns the complete set of businesses Google exposes, not just the first screen.

## What this actor does

- Covers a bounding box, circle, GeoJSON polygon, or named location with a grid of search zones.
- Reads each zone's complete result window (up to ~260 places across pages), not just the first 20.
- Collects rich listing data — category, address, rating, reviews, phone, website, hours, coordinates — without opening individual place pages.
- Streams accepted businesses into the dataset as the grid is covered, deduplicating overlapping zones by Place ID.
- Optionally opens each place page for extras: price level, Plus Code, photo sample, rating histogram, open/closed status.
- Empty fields are omitted.

## Output per business

With `enrichDetails=false` (the scalable default), each record contains everything Google's listing window exposes:

| Field | Description |
|---|---|
| `recordType` | Always `"business"`. |
| `name` | Business name. |
| `categories[]` / `categoryName` | All observed categories; the first is the primary one. |
| `address` | Full formatted address, when Google exposes one. |
| `totalScore` / `reviewsCount` | Displayed star rating and review total, when present. |
| `phone` / `website` | Public contact fields, when present. |
| `hours` | Weekly opening hours (`monday`…`sunday`), when Google exposes them; closed days read `Closed`, around-the-clock days read `Open 24 hours`. |
| `location` | Place centroid `lat`/`lng` — always the business location, never the map viewport. |
| `placeId` / `featureId` / `kgmid` | Canonical Place ID (`ChIJ…`), legacy feature ID, and Knowledge Graph ID. |
| `timezone` / `countryCode` | IANA timezone and ISO country of the place. |
| `rankInCell` | Position in the ranked window of the grid cell that found it (1 = top). |
| `googleMapsUrl` / `sourceUrl` | Stable Google Maps place URL. |
| `scrapedAt` | UTC extraction timestamp. |

With `enrichDetails=true`, records can additionally contain `plusCode`, `photoCount`, `photosSample` (up to ten public gallery images), `ratingHistogram` (1–5★ counts), `hoursStatus`, `isOpenNow`, `isPermanentlyClosed`, and `isTemporarilyClosed` — each only when Google publishes it. `priceLevel` is kept for compatibility but is almost never shown by Google Maps today.

## Input

You must provide **one** way to define the geographic area to scan:

- **Bounding box** (recommended) — `boundingBox` with `north`/`south`/`east`/`west`. Requires `south < north` and `west < east`; inverted or out-of-range coordinates are rejected.
- **Circle** — `circleCenter` (`lat`/`lng`) plus `radiusKm` (`0 < radiusKm ≤ 50`). The grid covers the circle's box; records are then checked against the exact great-circle radius.
- **GeoJSON polygon** — `customGeolocation`, a GeoJSON Polygon. Cells touching the outer ring are searched; records are checked against the exact boundary, with interior rings treated as holes.
- **Named location** — `locationName` (e.g. `"Brooklyn, NY"`). Google resolves the location's real center, then the actor scans `radiusKm` around it; use a bounding box or polygon when an exact boundary matters.

| Field | Type | Default | Description |
|---|---|---|---|
| `searchQuery` | String | – | **Required.** What to search for, e.g. `pharmacy`, `restaurant`, `hotel`. |
| `locationName` | String | – | Named public location to center the scan on (one of the four area options). |
| `boundingBox` | Object | – | Rectangular scan area (one of the four area options). |
| `circleCenter` | Object | – | Center point (`lat`/`lng`) for a circular scan area (one of the four area options). |
| `radiusKm` | Number | `5` | Radius in kilometers, used with `circleCenter` or `locationName` (0.1–50). |
| `customGeolocation` | Object | – | GeoJSON Polygon defining the scan boundary (one of the four area options). |
| `zoomLevel` | Integer | `15` | Grid cell granularity (12–18); higher means smaller cells and more thorough coverage. |
| `maxPlacesPerCell` | Integer | `120` | Maximum businesses collected from each cell (1–260). |
| `maxPagesPerCell` | Integer | `13` | Result pages read per cell (1–13). More pages surface more businesses — page one finds around 20% of a cell's places; all pages find all of them. |
| `maxTotalResults` | Integer | `10000` | Maximum businesses across all cells (1–100000). |
| `deduplicateResults` | Boolean | `true` | Remove duplicates that appear across adjacent cells. |
| `enrichDetails` | Boolean | `false` | Open each place page for the extra detail fields listed above. Slower. |
| `minRating` | Number | – | Only include businesses rated at or above this value (1.0–5.0); works in fast mode. |
| `language` | String (select) | `"en"` | Results language: English, Spanish, French, German, Italian, Portuguese, Japanese, Korean, Chinese, Arabic, or Russian. |

### Zoom level guide

| Zoom | Cell size | Best for |
|---|---|---|
| 12 | ~5 km | Large rural areas, quick overview |
| 13 | ~2.5 km | Suburban regions |
| 14 | ~1.25 km | Mixed urban/suburban |
| 15 (default) | ~625 m | Balanced coverage for most use cases |
| 16 | ~312 m | Dense urban areas |
| 17 | ~156 m | Very dense areas |
| 18 | ~78 m | Maximum detail |

### Example: bounding-box scan

```json
{
    "searchQuery": "coffee shops",
    "boundingBox": {"north": 40.76, "south": 40.73, "east": -73.97, "west": -74.02},
    "maxTotalResults": 1000,
    "enrichDetails": false
}
```

### Example: circle scan with detail enrichment

```json
{
    "searchQuery": "pharmacies",
    "circleCenter": {"lat": 40.7128, "lng": -74.0060},
    "radiusKm": 5,
    "enrichDetails": true
}
```

### Example: polygon scan

```json
{
    "searchQuery": "restaurants",
    "customGeolocation": {
        "type": "Polygon",
        "coordinates": [[[-74.02, 40.73], [-73.97, 40.73], [-73.97, 40.76], [-74.02, 40.76], [-74.02, 40.73]]]
    },
    "maxTotalResults": 5000
}
```

### Example: named location with a high rating filter

```json
{
    "searchQuery": "dentists",
    "locationName": "Austin, TX",
    "radiusKm": 10,
    "minRating": 4.5,
    "maxTotalResults": 500
}
```

## Use cases

- **Market mapping** — find every business of a given type across a whole city district or region.
- **Competitor density analysis** — map all restaurants, gyms, or clinics in a neighborhood to gauge saturation.
- **Site selection** — analyze business density and category mix before choosing a new location.
- **Lead generation** — collect contact details for every matching business across a target area.
- **Franchise/territory planning** — compare how many competing locations exist across candidate regions.
- **Local SEO research** — use `rankInCell` to see which businesses dominate each block of a search area.

## Tips for best results

- Start with a small test area to confirm your `searchQuery` returns the results you expect before scanning a large region.
- Zoom level 15 is a good balance of coverage and speed for most areas.
- Keep `deduplicateResults` on to avoid counting the same business twice across adjacent cells.
- Records appear in the dataset progressively as the scan runs, not only once the whole area finishes.
- Different phrasings of the same idea surface different businesses — run a synonym (e.g. `coffee` and `cafe`) and merge the datasets when you need maximum recall.

## Limitations

- Coverage per zone is limited by Google's ranked window: one zone query exposes roughly 140–210 distinct places at most, and a dense 625 m cell typically contributes around 10 new businesses. A 10,000-place export therefore needs many cells and can take a while.
- Very dense categories (e.g. every restaurant in Manhattan) need a finer zoom level: halving the cell size roughly doubled the unique places found in a dense 1 km² test (51 → 98), and a 4× finer grid with every page read found 179 (about 12× the zoom-15 requests). For a quick sweep stay at zoom 15; for maximum saturation use zoom 16–17 and keep all pages.
- Bounding-box, circle, and polygon scans require an observed place coordinate; a listing without one is omitted rather than assigned a guessed point.
- Fast mode has no Plus Code, photo gallery, photo count, rating histogram, or open/closed status — enable `enrichDetails` for those.
- `rankInCell` is the position in that scan's ranked window; Google's ordering shifts slightly between runs (about 60% of places stay within two positions on a repeat scan), so treat it as approximate rather than a stable rank metric.
- Coverage depends on what Google exposes for each zone at run time and can vary slightly between runs.

You are responsible for using the collected data lawfully and in line with Google's terms.

## FAQ

**Does this guarantee every business in an area?**
No. Coverage depends on what Google exposes for each zone, the chosen language, and run-time variation — but grid scanning recovers far more than a single capped search would.

**How should I scan a large area quickly?**
Leave `enrichDetails` off and leave `minRating` empty. Fast mode collects the full listing field set without opening any place page.

**Why do more pages mean more results?**
Google ranks each zone's results; a business deep in the ranking only appears on a later page. Page one holds about 20% of a cell's places, and the full window is reached around page nine, which is why the default reads every page. Finer cells also help, but only when every page is read: in a dense 1 km² test, zoom 15 found 51 unique places, zoom 16 found 98, and zoom 17 with all pages found 179 — at progressively higher request counts.

**Can it return more than 10,000 places?**
Yes — `maxTotalResults` accepts up to 100,000 and the scan continues across cells as long as Google keeps exposing real places. The practical limits are cell count and time.

**What happens when a grid cell is blocked or empty?**
The actor records only businesses actually observed in that cell and reports a truthful partial or complete result rather than padding the dataset with guesses.

**What's the difference between `placeId`, `featureId`, and `kgmid`?**
`placeId` is Google's canonical place token (`ChIJ…`). `featureId` is a legacy token (`0x…:0x…`) and is never relabeled as a Place ID. `kgmid` is the Knowledge Graph identifier (`/g/…`), when exposed.

**Do I need cookies or a login?**
No. The actor reads public Google Maps surfaces without any login session.

**Do I need to set up a proxy?**
No. The actor chooses a working connection to Google Maps automatically. Just fill in the input and run it.

**Is this affiliated with Google?**
No. This actor reads publicly available Google Maps pages and is not affiliated with or endorsed by Google.
