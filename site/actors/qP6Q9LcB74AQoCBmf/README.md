# Zillow County Home Value Trends — ZHVI 6 & 12-Month Change

Track typical US home values at county, state, and metro level from Zillow Research's monthly ZHVI files. Pick from ten housing cuts, compare true 1, 3, 6, and 12-month changes, rank markets, set a point-in-time as-of date, and see how a place is performing against its state or the nation. Every record is a flat row ready for spreadsheets, dashboards, and joins.

## What this actor does

- **Four modes:** one trend row per county (`countyTrends`), per state and DC (`stateTrends`), per metro and the US (`metroTrends`), or one tidy observation per region per month (`history`).
- **Ten housing cuts:** all homes mid-tier, single-family, condo/co-op, bottom tier, top tier, and 1 to 5+ bedrooms at every level Zillow publishes.
- **True month offsets:** 1, 3, 6, and 12-month values and percent changes use the same calendar month, not the nearest column.
- **Benchmarks:** counties are compared with their state, and states and metros with the United States, in percentage points.
- **Rankings:** sort by market size, name, value, any change window, or largest decline, and get a rank across every matching region before the row cap.
- **Point-in-time trends:** set `asOfDate` to reproduce a past month's view.
- **Join keys:** stable Zillow Region IDs plus state and county FIPS codes.
- **Clean output:** empty fields are omitted, never written as `null` or an empty string.

## Services

- **Zillow Research ZHVI:** the public Zillow Home Value Index files for county, state, and metro geographies, downloaded fresh on every run.
- **Statistics, not listings:** the actor returns published aggregate home-value statistics. It does not scrape property listings, agents, or Zillow pages.
- **No account or key:** the files are open downloads. No Zillow login, API key, cookie, or proxy configuration is needed.

## How it works

Each run reads the relevant Zillow Research file, finds the newest published value for each region (or the newest value on or before `asOfDate`), and looks up the exact month-end columns 1, 3, 6, and 12 months earlier. Percent changes are calculated from those matching months. A second file supplies the benchmark: the same housing cut for the state, or the United States row from the metro file. Matching regions are sorted, ranked, and capped at `maxItems`.

Everything is measured in US dollars for a typical home in the selected cut. Values come from the source file exactly as published; the only calculated fields are the change percentages, benchmark differences, and ranks. `scrapedAt` and `sourceUrl` record which release a row came from.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `countyTrends` | `countyTrends`, `stateTrends`, `metroTrends`, or `history`. The legacy values `search`, `countyData`, and `stateComparison` remain valid for old saved tasks and are mapped to the matching trend mode. |
| `homeType` | string | `allHomesMidTier` | Housing cut: `allHomesMidTier`, `singleFamily`, `condo`, `bottomTier`, `topTier`, `bedrooms1`, `bedrooms2`, `bedrooms3`, `bedrooms4`, `bedrooms5Plus`. |
| `state` | string | `all` | A state full name, two-letter code, or `all`. Filters county and metro rows and history, or selects one state in `stateTrends`. |
| `county` | string | – | Case-insensitive county-name substring (e.g. `Gallatin`, `Los Angeles`). Works in `countyTrends` and county history. |
| `regionIds` | array | – | Exact Zillow Region IDs such as `["3101", "1090"]`. Works in every mode and combines with other filters. |
| `asOfDate` | string | – | Trend as of the newest observation on or before `YYYY-MM-DD` (trend modes only). Empty uses the newest published month. |
| `historyGeography` | string | `county` | `county`, `state`, or `metro` — the level used by `history`. |
| `startDate` | string | – | First month for `history` (`YYYY-MM-DD`, inclusive). |
| `endDate` | string | – | Last month for `history` (`YYYY-MM-DD`, inclusive). |
| `sortBy` | string | `sizeRank` | `sizeRank`, `regionName`, `latestValue`, `change1mPct`, `change3mPct`, `change6mPct`, `change12mPct`, `decline6mPct`, or `decline12mPct`. |
| `maxItems` | integer | `100` | Hard cap on emitted rows (1–10,000). Ranking is computed before the cap. |

### Example: the largest six-month declines in the country

```json
{
  "mode": "countyTrends",
  "sortBy": "decline6mPct",
  "maxItems": 25
}
```

### Example: one county against its state

```json
{
  "mode": "countyTrends",
  "state": "Montana",
  "county": "Gallatin",
  "maxItems": 10
}
```

### Example: state trends with a past snapshot

```json
{
  "mode": "stateTrends",
  "asOfDate": "2025-09-30",
  "maxItems": 51
}
```

### Example: metro history for two Texas metros

```json
{
  "mode": "history",
  "historyGeography": "metro",
  "state": "Texas",
  "regionIds": ["394514", "394355"],
  "startDate": "2024-01-01",
  "endDate": "2026-09-30",
  "maxItems": 1000
}
```

### Example: every condo county in Colorado

```json
{
  "mode": "countyTrends",
  "homeType": "condo",
  "state": "Colorado",
  "maxItems": 100
}
```

## Output

### Trend records (`countyPropertyValue`, `statePropertyValue`, `metroPropertyValue`, `usPropertyValue`)

- `recordType` — which level the row describes.
- `geography`, `homeType` — `county` / `state` / `metro` / `us` and the selected housing cut.
- `regionId`, `sizeRank`, `regionName`, `regionType` — Zillow's stable region identity and market-size order.
- `countyName`, `stateCode`, `stateName`, `metro` — geography labels; county rows carry FIPS codes.
- `stateFips`, `countyFips` — county-level join keys.
- `latestValue`, `latestDate` — newest published ZHVI value and its month-end date.
- `value1MonthAgo`, `value3MonthsAgo`, `value6MonthsAgo`, `value12MonthsAgo` — same-month prior values.
- `change1mPct`, `change3mPct`, `change6mPct`, `change12mPct` — percent change from each prior month.
- `yearOverYearChangePct` — alias of `change12mPct`, kept for existing integrations.
- `benchmarkType`, `benchmarkName` — `state` for county rows, `us` for state and metro rows.
- `benchmarkLatestValue`, `benchmarkChange6mPct`, `benchmarkChange12mPct` — benchmark values in the same cut and as-of month.
- `vsBenchmark6mPct`, `vsBenchmark12mPct` — percentage-point difference from the benchmark.
- `rank`, `rankedBy` — 1-based rank across all matching regions and the chosen sort.
- `datasetType`, `propertyType`, `unit`, `currency` — human labels; `USD` for both unit and currency.
- `dataSource`, `sourceUrl`, `scrapedAt` — provenance for the exact Zillow Research release.
- `searchState` — the state filter you supplied, echoed on filtered trend rows.

### History records (`zhviObservation`)

- `recordType`, `geography`, `homeType`
- `regionId`, `sizeRank`, `regionName`, `regionType`
- `countyName`, `stateCode`, `stateName`, `metro`, `stateFips`, `countyFips`
- `date`, `value` — one month-end observation in USD.
- `unit`, `dataSource`, `sourceUrl`, `scrapedAt`

## Use cases

- Build a county-level market dashboard with six-month momentum and state context.
- Find the counties with the steepest six- or twelve-month declines in a selected housing cut.
- Compare single-family, condo, tier, and bedroom segments across states.
- Reproduce a prior month's ranking for reporting or backtesting with `asOfDate`.
- Pull tidy monthly series for charts, regressions, or joins against Census/FIPS data.
- Audit how one metro is tracking against the national trend.

## Usage cost

The actor adds no extra charge events of its own; you pay for Apify platform usage on your plan. A run reads one public CSV, or two when a benchmark file is needed. A full 3,071-county trend scan is about 14 MB of source data; history needs a date or region filter to stay under the 10,000-row cap.

## FAQ

**Is this actor affiliated with Zillow?**
No. It reads Zillow Research's openly published files and is not operated by or connected to Zillow. Please credit Zillow Research when you republish the data.

**Do I need a Zillow account, API key, or proxy?**
No. The ZHVI downloads are public and work without login, cookies, or proxy configuration.

**What exactly is ZHVI?**
The Zillow Home Value Index is a smoothed, seasonally adjusted estimate of the typical home value in a region for a chosen housing segment. It is a statistic, not a listing price or an appraisal.

**Why do the 6- and 12-month values differ from what I saw before?**
Older versions compared the latest column with the fifth- and eleventh-previous columns, which are really 5-month and 11-month comparisons. This version matches the same calendar month, so `change6mPct` is a true six-month change and `yearOverYearChangePct` is a true twelve-month change.

**Why is a 12-month change missing for a few counties?**
A handful of regions have a gap in the exact prior month. The field is omitted rather than interpolated or filled from a nearby month. Blaine County, Idaho is a known example.

**Why is `rank` missing on some rows?**
Rows without the metric you sorted by are placed last and receive no rank. All other rows are ranked across every matching region before `maxItems` cuts the list, so the top rows keep their true national, state, or metro rank.

**How current is the data?**
Zillow publishes monthly research files on a monthly cadence, usually mid-month for the prior month. Each run downloads the current release; `latestDate` shows the newest month in that file.

**Does Zillow ever revise past values?**
Yes. Zillow restates the history in each release. Treat a download as a snapshot of that release, not an immutable vintage; `scrapedAt` and `sourceUrl` identify it.

**Why does the number of counties differ between housing cuts?**
Zillow only publishes a region in a cut when it has enough data for that cut. Condos and bedrooms have fewer regions than all homes, and state coverage is always 50 states plus DC.

**Can I get more than 10,000 history rows?**
Yes, by splitting the request. Filter to one state or a set of `regionIds`, or narrow `startDate` / `endDate`. The status message tells you when a run stopped at the cap.

## Notes

- All values are monthly USD statistics for the selected cut. Month-end dates are kept exactly as published.
- The source covers 50 states plus the District of Columbia; DC is available in state trends and history.
- County records include state and county FIPS codes. Use `regionIds` when a county name appears in several states.
- Benchmarks use the same housing cut and the same as-of month as the row being compared.
- `yearOverYearChangePct` is retained for backward compatibility and always equals `change12mPct`.
- Records are flat and omit empty fields, so downstream code should treat absent optional fields as "not published for that region or month".
- The status message reports the mode, housing cut, geography, rows scanned, rows emitted, as-of date, benchmark source, and whether the row cap truncated the result.

## Limitations

- Zillow's county universe is smaller than the official US county count: the all-homes file has 3,071 counties. Missing counties are a source coverage limit, not a filter the actor can relax.
- Puerto Rico and US territories are not in these ZHVI files.
- County-to-metro benchmarks are not offered: the county file uses full CBSA names while the metro file uses short Zillow names, and there is no shared metro ID. County-to-state and state/metro-to-US benchmarks are available.
- The 1996+ raw metro series is not exposed; the ten selectable cuts use the smoothed, seasonally adjusted files that start in 2000.
- History is capped at 10,000 rows per run and always tells you when it truncated.
- A few regions lack an exact 1, 3, 6, or 12-month prior. Those values and changes are omitted, never interpolated.
- Zillow restates its history each release, and the source updates monthly rather than in real time.
- Zillow can rename or retire a research file. The actor uses a verified static file map; if a future release changes a filename, the affected run reports a fetch failure instead of returning fabricated data.
- Home values, rents, inventory, forecasts, and market heat are outside this actor's scope; it focuses on ZHVI trends and comparisons.
