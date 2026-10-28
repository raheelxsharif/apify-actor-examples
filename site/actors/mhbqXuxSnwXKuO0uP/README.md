# Zillow Research Home Value & Market Heat Scraper

Zillow publishes state and metro-level housing statistics as open research data. This actor turns those files into dataset records: state home value trends, active-listing inventory, and metro market heat index data, each filterable by state.

## What this actor does

- Returns recent monthly state-level home value data (the Zillow Home Value Index), optionally filtered to one state.
- Falls back to active-listing inventory data for the requested state when no home value rows match.
- Returns metro-level market heat index data for market-temperature research.
- Caps every run at a configurable number of records.
- Leaves out a region's monthly value columns when Zillow has no published figure for that month, rather than sending an empty value.

## Output per record

- `regionName` — the state, county, or metro area name
- `regionType` — the region level (e.g. state, county, or metro/msa)
- `stateName` — the state the region belongs to, when published
- `metro` — the metro area name, when the region is a metro
- `value_<YYYYMMDD>` — one field per recent month with a published value for that date (field names change as new months are published)
- `latestValue`, `latestDate` — the most recent monthly value and its date
- `datasetType` — human-readable label for the dataset this record came from (`Home Values (ZHVI)`, `Active Listings Inventory`, or `Market Heat Index`)
- `propertyType` — property segment covered (`Single Family / Condo`), present on home value records
- `currency` — always `USD`, present on home value and inventory records
- `dataSource` — always `Zillow Research`
- `sourceUrl` — the Zillow Research file the record was read from
- `recordType` — `propertyValue`, `propertyInventory`, or `marketTrend`
- `scrapedAt` — UTC timestamp of the fetch

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `search` | `search` and `stateValues` both return state home value data (with an inventory fallback); `nationalTrends` returns metro market heat data |
| `state` | string | `Texas` | US state name to filter results to |
| `datasetType` | string | `homeValues` | `homeValues`, `inventory`, `marketHeat`, or `countyHomeValues`; the active `mode` currently determines which dataset is actually used (see above) |
| `maxItems` | integer | `20` | Maximum number of records to return (1-500) |

### Example: home values for one state

```json
{
  "mode": "search",
  "state": "Texas",
  "maxItems": 10
}
```

### Example: home values for every state

```json
{
  "mode": "stateValues",
  "maxItems": 50
}
```

### Example: metro market heat index

```json
{
  "mode": "nationalTrends",
  "maxItems": 100
}
```

## Use cases

- Track a state's recent home value trend without visiting Zillow's research pages.
- Compare active-listing inventory across states.
- Pull metro-level market heat data for a housing market report.
- Feed home value history into a broader real-estate or economic dataset.

## FAQ

**Is this affiliated with Zillow?**
No. This actor reads Zillow's own published research files; it isn't operated by or connected to Zillow.

**Do I need a Zillow account or API key?**
No. Zillow publishes these research files openly, with no login required.

**Why does `state` not narrow down `nationalTrends` results?**
`nationalTrends` returns metro-area rows (e.g. "Dallas-Fort Worth, TX"), not state rows, so a state name match against those regions will often return nothing; `state` is most useful with `search` or `stateValues`.

**Why do I sometimes get inventory records instead of home values?**
If no home value rows match your `state` filter, the run automatically returns active-listing inventory data for that state instead of an empty result.

**Why do the `value_<date>` field names change between runs?**
Each one reflects an actual month Zillow has published a figure for; as Zillow adds new months, the most recent twelve column names shift forward.

**How many records can one run return?**
Up to `maxItems`, capped at 500 per run.

**How current is the data?**
Each run downloads Zillow's current published research file, so results reflect whatever Zillow has most recently released.
