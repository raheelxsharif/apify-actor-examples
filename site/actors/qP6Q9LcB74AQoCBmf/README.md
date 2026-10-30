# Zillow Research County Home Value Trend Scraper

Zillow publishes county and state housing statistics as open research data, including how home values have moved over the past year. This actor turns those files into dataset records for a specific state/county or for every US state at once, each one carrying a 6-month and 12-month comparison alongside the latest value.

## What this actor does

- Returns recent county-level home value data (the Zillow Home Value Index), optionally filtered to a state and/or county.
- Falls back to state-level home value data when no county rows match your filter.
- Returns a full state-by-state home value comparison in `stateComparison` mode.
- Calculates the percent change between the latest value and the value from roughly 12 months earlier, when both are available.
- Caps every run at a configurable number of records.
- Omits a comparison field (6-month value, 12-month value, year-over-year change) when Zillow's published history isn't long enough to calculate it.

## Output per record

- `countyName`, `stateCode`, `metro` — present on county-level records
- `stateName` — present on state-level records (the fallback and `stateComparison` mode)
- `searchState` — the state filter you applied, echoed back on county records
- `latestValue`, `latestDate` — the most recent published monthly value and its date
- `value6MonthsAgo`, `value12MonthsAgo` — values from roughly 6 and 12 months earlier, when published
- `yearOverYearChangePct` — percent change between `latestValue` and `value12MonthsAgo`, when both exist
- `datasetType` — human-readable label (`County Home Values (ZHVI)` or `State Home Values (ZHVI)`)
- `propertyType` — `Single Family / Condo`, present on county-level records
- `currency` — always `USD`
- `dataSource` — always `Zillow Research`
- `sourceUrl` — the Zillow Research file the record was read from
- `recordType` — `countyPropertyValue` or `statePropertyValue`
- `scrapedAt` — UTC timestamp of the fetch

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `search` | `search` and `countyData` both return county-level data (with a state-level fallback); `stateComparison` returns every state's home value data |
| `state` | string | `Montana` | US state name to filter county results to; used by `search`/`countyData` |
| `county` | string | – | County name substring to filter to, e.g. `Gallatin`; used by `search`/`countyData` |
| `maxItems` | integer | `20` | Maximum number of records to return (1-500) |

### Example: one county's home value trend

```json
{
  "mode": "search",
  "state": "Montana",
  "county": "Gallatin",
  "maxItems": 10
}
```

### Example: every county in a state

```json
{
  "mode": "countyData",
  "state": "Texas",
  "maxItems": 100
}
```

### Example: state-by-state comparison

```json
{
  "mode": "stateComparison",
  "maxItems": 50
}
```

## Use cases

- Track a specific county's home value trend and recent percent change.
- Compare home values across every county in a state.
- Rank states by home value or year-over-year change for a market overview.
- Feed county-level value history into a broader real-estate research dataset.

## FAQ

**Is this affiliated with Zillow?**
No. This actor reads Zillow's own published research files; it isn't operated by or connected to Zillow.

**Do I need a Zillow account or API key?**
No. Zillow publishes these research files openly, with no login required.

**Why did I get state-level records when I filtered by county?**
If no county rows match your `state`/`county` filter, the run automatically falls back to state-level home value data for that state so you still get a result.

**Why is `yearOverYearChangePct` missing from some records?**
It's only calculated when both the latest value and a value from roughly 12 months earlier are published; counties or states with a shorter published history won't have it.

**What's the difference between `search`, `countyData`, and `stateComparison`?**
`search` and `countyData` behave the same way — both return county-level data, filtered by `state`/`county` when provided. `stateComparison` ignores the county filter and returns one record per state instead.

**How many records can one run return?**
Up to `maxItems`, capped at 500 per run.

**How current is the data?**
Each run downloads Zillow's current published research file, so results reflect whatever Zillow has most recently released.
