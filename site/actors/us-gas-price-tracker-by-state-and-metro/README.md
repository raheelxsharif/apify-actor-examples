# US Gas Price Tracker by State & Metro

Get daily US gas and diesel prices without opening a fuel-price site yourself. This actor reads the public AAA Fuel Gauge data and returns it as structured records at whichever level you need: the national average, all 50 states plus DC, or the metro/county breakdowns inside specific states.

## What this actor does

- **Three scopes:** one national record, the state table (50 states + DC), or metro/county averages within chosen states
- **Five fuel grades:** regular, mid-grade, premium, diesel, and E85. E85 is published by AAA only for the national average, so it is omitted from state and metro records rather than sent as a null
- **Published price date:** every record carries `priceDate`, the "Price as of" date AAA prints on the page it came from
- **Historical comparison:** yesterday, week-ago, month-ago, and year-ago prices alongside today's, plus derived cent-value changes for a chosen primary fuel
- **Filters:** by state, by fuel grade, by a price band on the primary fuel, and by metro-name substring
- **Pagination:** read a large result set in pages with `startIndex` and `maxItems`
- **Empty fields are omitted** — a record only includes the grades and history buckets AAA actually published for it

## Output per record

- `scope` — `national`, `state`, or `metro`
- `country` — always `US`, present only on the national record
- `state`, `stateCode` — state name and two-letter code, present on state and metro records
- `metro` — metro/county area name, present only on metro records
- `priceDate` — the AAA "Price as of" date in `YYYY-MM-DD` form
- `regular`, `midGrade`, `premium`, `diesel` — current price in USD/gallon for each grade the source publishes
- `e85` — current E85 price in USD/gallon, national record only
- `history` — an object with `yesterday`, `weekAgo`, `monthAgo`, and `yearAgo` prices per fuel grade, when `includeHistory` is enabled and AAA publishes them
- `changeFromYesterday`, `changeFromWeekAgo`, `changeFromMonthAgo`, `changeFromYearAgo` — price change in cents for the chosen `primaryFuelType`, when `includeChangeMetrics` is enabled and the matching history value exists
- `recordType` — always `fuelPrices`
- `scrapedAt` — when the record was fetched

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `state` | `national` (1 record), `state` (50 + DC), or `metro` (per-state metro/county averages) |
| `states` | array | `[]` | Two-letter codes or names (e.g. `NY`, `CA`, `New York`). Empty includes all 50 states + DC. For `mode=metro`, this also selects which state pages are read |
| `fuelTypes` | array | `["regular", "midGrade", "premium", "diesel", "e85"]` | Which fuel grades to keep in each record. A record is dropped if none of the requested grades exist on it |
| `minPrice` | number | – | Drop records whose primary fuel is priced below this value |
| `maxPrice` | number | – | Drop records whose primary fuel is priced above this value |
| `primaryFuelType` | string | `regular` | Which fuel grade `minPrice`, `maxPrice`, `sortBy`, and the change fields use |
| `metroNameFilter` | array | `[]` | Only emit metro records whose name contains one of these substrings (mode=metro only) |
| `includeHistory` | boolean | `true` | Include yesterday/week-ago/month-ago/year-ago prices. In state mode this reads each selected state's page, so the prefilled daily run turns it off |
| `includeChangeMetrics` | boolean | `true` | Compute the four `changeFrom*` fields in cents, when history is available |
| `sortBy` | string | `state_asc` | `price_asc`, `price_desc`, or `state_asc` |
| `maxItems` | integer | `500` | Records per page (1–5000) |
| `startIndex` | integer | `0` | How many matching records to skip before this page (0–5000) |

### Example: the state table, one request

```json
{
  "mode": "state",
  "includeHistory": false,
  "sortBy": "price_asc",
  "maxItems": 10
}
```

### Example: page two of all states, cheapest first

```json
{
  "mode": "state",
  "includeHistory": false,
  "sortBy": "price_asc",
  "maxItems": 10,
  "startIndex": 10
}
```

### Example: metro breakdown for two states

```json
{
  "mode": "metro",
  "states": ["NY", "CA"],
  "metroNameFilter": ["new york", "los angeles", "san francisco"]
}
```

### Example: the national average with E85 and history

```json
{
  "mode": "national",
  "includeHistory": true,
  "primaryFuelType": "e85"
}
```

## How pagination works

The actor first applies every filter and sort, then slices the matching records with `startIndex` and `maxItems`. The run status message reports how many records matched, how many this page emitted, and the `startIndex` for the next page when more records are available. Pagination is over the deterministic result set, so repeating the same input returns the same page.

## Legacy inputs

Saved tasks that use older or alternative key names keep working because the mapping lives in code, not in the input form: `level`, `geography`, `scope`, `state`, `fuelType`, `fuel`, `fuels`, `includeChange`, `history`, `sort`, `limit`, `maxResults`, `offset`, and `metroNames` are accepted and mapped onto the current fields. A legacy key overrides a new field that still holds its schema default, so the defaults the platform pre-fills do not mask an old saved task. New runs should use the fields shown in the input table.

## Use cases

- **Fleet and logistics planning** — compare diesel prices across states before routing
- **News and editorial research** — pull a "cheapest gas by state" snapshot on demand
- **Consumer price widgets** — feed a national or regional average into a dashboard
- **Trend monitoring** — watch `changeFromYearAgo` to catch large year-over-year price swings
- **Economic analysis** — fuel prices as an input to broader cost-of-living or inflation tracking

## FAQ

**Where does this data come from?**
The public AAA Fuel Gauge pages, the same averages AAA publishes for national, state, and metro-level gas prices.

**Do I need a login, cookies, or a proxy?**
No. The source pages are fully public and the actor reads them without authentication. The actor does not need a proxy configuration.

**How current is the data?**
AAA refreshes its averages daily. Each run reads the page live, and `priceDate` tells you the exact date AAA printed, so you can see when AAA had not updated yet.

**Why is E85 only on national records?**
AAA's national table includes an E85 column, but the state and metro tables do not. The actor omits the field there instead of inventing a value. Setting `fuelTypes` to `["e85"]` in state or metro mode therefore returns no records.

**Why might a record be missing a fuel grade?**
Not every state or metro area has a published price for every grade (mid-grade and premium especially). The actor omits a grade rather than sending a null or zero value.

**Why is a change metric like `changeFromMonthAgo` sometimes absent?**
It is only computed when both the current price and the matching history value are available for that record's primary fuel.

**What does `metroNameFilter` match against?**
A case-insensitive substring match against the metro/county area name as published by AAA (e.g. `Albany-Schenectady-Troy`, `Nassau-Suffolk`).

**Why does metro mode take longer for many states?**
AAA's robots.txt asks for 10 seconds between requests. The actor honours that interval while reading state pages, so a two-state metro run takes about 20 seconds and a full 51-state run takes several minutes. State mode with `includeHistory` turned off needs just one request.

**Why can't I get individual station prices?**
AAA publishes state and metro averages, not per-station prices, so station-level data is outside this source. The actor reports the granularity AAA actually provides.

**Does this cover countries outside the US?**
No, this is US-only; the underlying source does not publish averages for other countries.
