# US Walkability, Transit & Bike Score Extractor

Look up **Walk Score**, **Transit Score**, and **Bike Score** for US addresses, cities, neighborhoods, and states. Each record carries the 0–100 scores with the rating labels and summaries shown on walkscore.com, plus coordinates, city/state/ZIP, transit lines, nearby parks, and city or neighborhood context. No API key, login, or proxy setup is needed.

## What this actor does

- **Five modes:** `byAddress` (bulk), `byCity` (with an optional neighborhood ranking table), `byNeighborhood`, `byState` (all cities in one state), and `byRanking` (top US cities).
- **Correct ratings:** Walk/Transit/Bike labels and summaries are read from each page, so low-score categories such as "Car-Dependent" and "Somewhat Bikeable" are labelled correctly.
- **Location context:** latitude/longitude, city, state, ZIP, transit lines with type and distance, nearby park names, city averages, population, and walkability ranks where the page shows them.
- **Bulk-friendly:** one address per lookup, or a whole city/state table in a single request.
- **Archive fallback:** if walkscore.com is unavailable, the actor can return a recent Wayback Machine snapshot with explicit provenance. City and neighborhood pages are more likely to be archived than arbitrary addresses.
- **Empty fields are omitted.** A location without a Transit Score is not given a zero; the field is simply absent.

### Archive data limitations

Wayback snapshots are a best-effort fallback, not a live feed. Archive records are marked with
`dataSource: "wayback"` and carry the snapshot time and URL. Data can be days or weeks old and
scores are point-in-time. City and neighborhood pages are archived more often than arbitrary
street addresses; when no snapshot exists, the actor says so instead of inventing a score.

## Output per location

- `locationType` — `address`, `city`, `neighborhood`, `state`, or `error`
- `address` — the canonical display label from the page
- `inputAddress`, `addressMatched` — the submitted address and whether the geocoder kept the same street (address records)
- `city`, `state`, `zipCode`, `country`
- `latitude`, `longitude` — decimal degrees
- `walkScore`, `walkDescription`, `walkSummary`
- `transitScore`, `transitDescription`, `transitSummary`
- `bikeScore`, `bikeDescription`, `bikeSummary`
- `transitLines[]` — `type` (`rail`/`bus`), `name`, `distanceMiles`
- `nearbyAmenities[]` — nearby park names the page lists
- `cityAverageWalkScore`, `cityPopulation` — city context on address pages
- `population`, `walkRankUS`, `transitRouteCounts`, `topNeighborhoods`, `neighborhoods[]` — city/neighborhood context
- `stateName`, `stateAverageWalkScore`, `stateCityCount`, `bestCities`, `worstCities` — state summary
- `sourceUrl`, `recordType`, `scrapedAt`
- `dataSource` — `live` or `wayback` (archived fallback)
- `archivedAt` — ISO timestamp of the Wayback snapshot (archive records only)
- `archiveUrl` — the raw Wayback snapshot URL (archive records only)
- `status` — provenance note, e.g. `live` or `archived snapshot; live source unavailable (HTTP 503)`
- `_message` — present only on a diagnostic record when neither live nor archived data is available

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `byCity` | `byAddress` / `byCity` / `byNeighborhood` / `byState` / `byRanking` |
| `addresses` | array | `[]` | US addresses, e.g. `1 Beach St, San Francisco, CA 94133` (mode=byAddress) |
| `cities` | array | `["Seattle, WA"]` | `Seattle, WA`, `Seattle-WA`, or a city URL (mode=byCity) |
| `neighborhoods` | array | `[]` | `Ballard, Seattle, WA` or a neighborhood URL (mode=byNeighborhood) |
| `state` | string | `WA` | US state code (mode=byState) |
| `includeNeighborhoods` | boolean | `true` | Add the city page's neighborhood table (mode=byCity) |
| `maxItems` | integer | `50` | Hard cap, 1–1000 (the state summary counts as one) |

### Example: score a list of addresses

```json
{
  "mode": "byAddress",
  "addresses": [
    "1 Beach St, San Francisco, CA 94133",
    "1519 8th Ave, Seattle, WA 98101",
    "County Road 15, Johnstown, CO 80534"
  ],
  "maxItems": 10
}
```

### Example: cities with their neighborhood table

```json
{
  "mode": "byCity",
  "cities": ["Seattle, WA", "Chicago, IL"],
  "includeNeighborhoods": true,
  "maxItems": 5
}
```

### Example: every city in a state

```json
{
  "mode": "byState",
  "state": "WA",
  "maxItems": 100
}
```

### Example: a neighborhood or the national ranking

```json
{"mode": "byNeighborhood", "neighborhoods": ["Ballard, Seattle, WA"], "maxItems": 3}
```

```json
{"mode": "byRanking", "maxItems": 25}
```

## Use cases

- **Relocation research** — compare walk, transit, and bike access across candidate addresses and cities.
- **Real-estate enrichment** — attach scores and neighborhood context to property lists by address or ZIP-level city.
- **Site selection** — screen retail/office addresses by walkability and transit access.
- **Urban planning and research** — pull a whole state's city table or a city's neighborhood ranking in one run.
- **Content and comparison sites** — build city and neighborhood rankings from live Walk Score data.
- **Data pipelines** — normalize addresses and merge scores with coordinates, city/state/ZIP, and transit context.

## FAQ

**Do I need a Walk Score API key?**
No. This actor reads walkscore.com's public pages. The official Walk Score API is not used.

**Why is a Transit Score or Bike Score missing for some locations?**
walkscore.com only publishes those scores where it has enough data. Rural and exurban locations often have no Transit Score; the field is omitted rather than set to zero.

**Why does `nearbyAmenities` only contain park names?**
The public pages name nearby parks, and list transit lines separately. They do not publish restaurant, grocery, or coffee-shop names or distances; those are not invented.

**My address changed after lookup. Why?**
walkscore.com's geocoder can snap a vague or invalid address to the nearest known address. Compare `inputAddress` with `address` and check `addressMatched`; the canonical `sourceUrl` is always included.

**Can I get the neighborhood name for a street address?**
Not yet. Address pages give city, state, ZIP, and coordinates. City records include a full neighborhood table with scores and population, and `byNeighborhood` looks up a named neighborhood directly.

**What happens if walkscore.com is unavailable?**
The actor tries the live page first. If the site is down, it looks for a recent Wayback Machine snapshot and returns it with `dataSource: "wayback"`, `archivedAt`, `archiveUrl`, and a `status` note. Archive data can be days or weeks old, so scores are point-in-time; city and neighborhood pages are more likely to be archived than arbitrary addresses. If neither live nor archived data exists, a diagnostic record with `_message` is returned instead of silently returning no rows.

**Does this work outside the US?**
It is built for US city, neighborhood, address, and state pages. Canadian and other international pages are not covered.

**Is this affiliated with Walk Score or Redfin?**
No. This is a third-party actor reading publicly available pages. Walk Score is a trademark of Redfin Corporation.

**Do I need to configure a proxy?**
No. The actor chooses a working connection automatically.
