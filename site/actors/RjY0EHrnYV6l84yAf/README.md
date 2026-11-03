# WTA Washington Trails, Hike Details & Trip Reports

Scrape **Washington Trails Association (wta.org)** — the state's largest hiking guide — with four modes covering trail search, trail details, trail-level trip reports and report-level search. Pull all 4,278 hikes across 11 regions and 51 sub-regions, with length, elevation, coordinates, difficulty, access rules, parking passes, alerts and recent trail conditions. No account, cookies or proxy setup required.

## What this actor does

- **Four modes:** `searchTrails`, `getTrailDetails`, `getTripReports`, `searchTripReports`
- **Complete trail inventory:** all 4,278 WTA hikes, 11 regions and 51 sub-regions, paginated in one run
- **Hiking filters:** region, sub-region, name/full-text search, features, dog policy, kid/wheelchair access, rating, votes, length, gain, high point, required pass, report recency and seven sort orders
- **Map search:** radius around a latitude/longitude or a `south,west,north,east` bounding box
- **Real trip reports:** trail-level report counts and latest dates, or a filtered feed from 288,600 reports with conditions, photos, highlights and helpful votes
- **Detail enrichment:** difficulty, parking pass, land manager, directions, closures and full description
- **Dual units:** miles/feet from the source plus derived kilometres/metres
- **Empty fields are omitted** — records never contain `null`, empty strings or empty arrays

## Output per trail

- `recordType` — `"trail"`
- `trailName`, `trailUrl`, `trailId`, `hikeUid`, `trailSlug`
- `region`, `regionName`, `subregionName`
- `lengthMiles`, `lengthText`, `lengthKm`, `routeType`
- `elevationGainFeet`, `elevationGainText`, `elevationGainMeters`
- `highPointFeet`, `highPointText`, `highPointMeters`
- `difficulty` — Easy through Hard (detail mode or `fetchDetails`)
- `rating`, `reviewCount` — community rating and number of rating votes
- `tripReportCount`, `lastTripReportDate`, `lastTripReportUrl`
- `features[]` — Lakes, Waterfalls, Mountain views, Summits and other WTA tags
- `latitude`, `longitude`, `distanceMiles` (radius searches)
- `parkingPass`, `passes[]`, `landManager`, `directions`
- `alerts[]`, `hasAlerts` — closures and access warnings checked with the trail
- `companions[]`, `dogsAllowed`, `dogPolicy`, `kidFriendly`, `wheelchairAccess`
- `summary`, `description`, `imageUrl`
- `scrapedAt` — ISO-8601 UTC timestamp

## Output per trip report

- `recordType` — `"tripReport"`
- `tripReportId`, `tripReportUrl`
- `trailName`, `trailUrl`, `trailId`, `hikeUid`
- `reportDate`, `region`, `regionName`, `subregionName`
- `author`, `authorUrl`, `authorBadges[]`
- `text`, `photoCount`, `imageUrl`, `photoUrls[]`
- `helpfulCount`, `features[]`
- `hikeType`, `trailConditions`, `roadConditions`, `bugs`, `snow`
- `warnings`, `scrapedAt`

`getTripReports` returns trail records with `tripReportCount`, `lastTripReportDate`, `lastTripReportUrl` and a `recentTripReports[]` array containing each report's date, author, URL, warnings and optional text/tags when `includeReportText` is on.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | select | `searchTrails` | `searchTrails`, `getTrailDetails`, `getTripReports` or `searchTripReports` |
| `searchQuery` | string | – | Full-text search of hike descriptions (searchTrails, searchTripReports) |
| `nameQuery` | string | – | Match the hike name only (searchTrails, getTripReports, searchTripReports) |
| `region` | select | – | All Washington or one of 11 regions |
| `subregion` | select | `all` | Any of 51 sub-regions; also selects its parent region |
| `features` | array | `[]` | Require all selected trail features (searchTrails) |
| `dogPolicy` | select | `any` | `any`, `allowed-on-leash` or `not-allowed` |
| `kidFriendly` | boolean | `false` | Only kid-friendly trails |
| `wheelchairFriendly` | boolean | `false` | Only wheelchair-friendly trails |
| `minRating` | number | – | Minimum average rating (0–5) |
| `minVoteCount` | integer | – | Minimum number of rating votes |
| `minLengthMiles` / `maxLengthMiles` | number | – | Trail length range (miles) |
| `minElevationGainFeet` / `maxElevationGainFeet` | integer | – | Elevation gain range (feet) |
| `maxHighPointFeet` | integer | – | Maximum highest point (feet) |
| `passes` | array | `[]` | Keep trails requiring at least one selected pass |
| `reportRecency` | select | `anytime` | `anytime`, `30-to-90-days` or `90-plus-days` |
| `sortBy` | select | `recently-updated` | Recently updated, rating, difficulty, distance, gain, high point or name |
| `latitude`, `longitude`, `radiusMiles` | numbers | – | Radius search; all three are required together |
| `boundingBox` | string | – | `south,west,north,east`; takes priority over radius |
| `trailUrls` | array | `[]` | WTA URLs, `/go-hiking/hikes/<slug>` paths or bare slugs |
| `fetchDetails` | boolean | `false` | Add detail-page fields per trail |
| `author` | string | – | Report author username (searchTripReports) |
| `tripDateFrom` / `tripDateTo` | string | – | Report date range, YYYY-MM-DD |
| `tripMonth` | select | `all` | All months or January–December |
| `hikeTypes` | array | `[]` | Day hike, overnight, multi-night backpack, snowshoe/XC ski |
| `reportFeatures` | array | `[]` | Has photos, hiked by kids, hiked with a dog, work party |
| `reportHighlights` | array | `[]` | Wildflowers, fall foliage, ripe berries |
| `trailConditions` | array | `[]` | Good, difficult, bugs okay, snow free, snow on trail |
| `roadConditions` | array | `[]` | All vehicles, high clearance, impassable |
| `fetchReportDetails` | boolean | `false` | Open each report page for conditions and photo URLs |
| `tripReportsPerTrail` | integer | `5` | Recent reports per trail (getTripReports) |
| `includeReportText` | boolean | `false` | Attach text and tags to recent reports |
| `maxItems` | integer | `50` | Hard cap on emitted records (1–10,000) |
| `proxyConfiguration` | object | – | Optional override; the actor works without a proxy |

### Example: browse a region with hiking filters

```json
{
  "mode": "searchTrails",
  "region": "Snoqualmie Region",
  "features": ["Waterfalls", "Lakes"],
  "minLengthMiles": 3,
  "maxLengthMiles": 10,
  "minRating": 4,
  "maxItems": 40
}
```

### Example: full details for specific trails

```json
{
  "mode": "getTrailDetails",
  "trailUrls": [
    "https://www.wta.org/go-hiking/hikes/mount-si",
    "snow-lake-1"
  ]
}
```

### Example: reports near a map point

```json
{
  "mode": "searchTrails",
  "latitude": 47.4880,
  "longitude": -121.7231,
  "radiusMiles": 25,
  "maxItems": 25
}
```

### Example: recent trip reports with conditions

```json
{
  "mode": "searchTripReports",
  "region": "Snoqualmie Region",
  "tripDateFrom": "2026-09-01",
  "reportFeatures": ["has-photos"],
  "fetchReportDetails": true,
  "maxItems": 30
}
```

### Example: report history for one trail

```json
{
  "mode": "getTripReports",
  "trailUrls": ["https://www.wta.org/go-hiking/hikes/mount-si"],
  "tripReportsPerTrail": 10,
  "includeReportText": true
}
```

## Use cases

- **Trip planning** — shortlist hikes by region, distance, gain, rating, dog rules and required pass
- **Outdoor publishing** — build a Washington hiking guide with descriptions, difficulty and photos
- **Trail-condition monitoring** — track recent reports, snow, road access and closure alerts
- **Mapping and GIS** — export trailhead coordinates for every trail that publishes a location
- **Recreation research** — compare trail density, length and elevation across 11 regions
- **Trip-report analysis** — study seasonality, highlights and helpful votes across 288,600 reports

## FAQ

**Is this affiliated with the Washington Trails Association?**
No. This is an independent, third-party actor that reads publicly available pages on wta.org. It is not operated by or endorsed by WTA.

**What is the difference between the four modes?**
`searchTrails` browses or filters the full trail database. `getTrailDetails` returns one complete record for each WTA URL or slug you paste. `getTripReports` returns trails with report counts, latest report dates and recent report metadata. `searchTripReports` returns individual reports from the global report feed.

**How many trails and reports are available?**
WTA currently lists 4,278 hikes in 11 regions and 51 sub-regions. The global report feed contains about 288,600 reports. A full trail run returns the complete inventory; a full report run is capped by `maxItems` (up to 10,000 per run).

**Why are `lengthMiles`, `rating`, coordinates or features missing on some trails?**
Roughly one fifth to one quarter of WTA records genuinely do not publish every field, and about 620 trails have no coordinates. Those fields are omitted rather than filled with guesses. Filters that need a missing length, elevation, rating, feature or coordinate exclude that record; WTA's own server-side filters enforce dog, kid and wheelchair access.

**What is the difference between `reviewCount` and `tripReportCount`?**
`reviewCount` is the number of community rating votes behind `rating`. `tripReportCount` is the separate count of trip reports published for the trail.

**How do I get coordinates, difficulty and access details?**
Set `fetchDetails` to `true` in `searchTrails` or use `getTrailDetails`. Search results already include coordinates and access attributes for the 85% of trails that publish them in WTA's map data; difficulty, parking pass, land manager, directions and alerts come from the trail page.

**Do I need an account, cookies or a proxy?**
No. The actor uses public WTA pages and works without credentials or proxy setup. `proxyConfiguration` is an optional override only.

**Why does a report search return different reports than a trail's report history?**
`searchTripReports` reads WTA's global report feed and applies report-level filters such as author, month, hike type and conditions. `getTripReports` reads each selected trail's own report history, which is the better choice when you care about one trail.

**How current is the data?**
Each run reads the live WTA pages. Trail stats, report counts and report dates reflect the site at run time. Hikers post new reports continuously, so recent-report fields change from day to day.

**Can I get every trail in one run?**
Yes. Run `searchTrails` with `region` set to All Washington and `maxItems` at least 4,278. Region-by-region runs work too if you want smaller batches.
