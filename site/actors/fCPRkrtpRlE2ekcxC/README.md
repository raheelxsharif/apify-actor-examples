# Trading Economics Markets, Indicators & Calendar Scraper

Trading Economics publishes live market quotes, macro indicators, forecasts, release calendars and financial news for nearly every country. This actor turns those public pages into clean JSON: live quotes, country indicators, global rankings, economic calendar events with actual/consensus/revision values, quarterly forecasts, holidays, earnings, long historical series, market forecasts, news, keyword search, credit-rating history and full government-bond curves.

## What this actor does

- **14 modes:** `markets`, `countryIndicators`, `indicatorRanking`, `calendar`, `countryForecast`, `holidays`, `earningsCalendar`, `indicatorHistory`, `marketHistory`, `marketForecast`, `news`, `search`, `creditRatingHistory`, `bondYields`
- **Multi-country and multi-indicator:** pass several countries, ranking indicators or symbols in one run; the item cap is spread round-robin so each requested target is represented
- **Long history:** economic indicators and market prices from one month to the full available history, including intraday intervals
- **Calendar precision:** numeric actual/previous/consensus/forecast values, revision markers and a derived surprise when the units match
- **Everything else:** global rankings, quarter-ahead country projections, public holidays, global or per-country earnings, news stream, symbol search, rating histories and every bond maturity
- **Empty fields are omitted** — no nulls, empty strings or placeholder values anywhere in the output

## Services and data covered

| Service | What you get |
|---|---|
| Markets | Commodities, FX pairs, stock indices, 10Y bond yields, crypto and top-moving shares |
| Macro indicators | GDP, labour, prices, money, trade, government, business, consumer, housing, energy and health tables by country |
| Global rankings | Any of 163 indicator slugs ranked across all reporting countries, including sovereign credit ratings |
| Economic calendar | Global or filtered by country with the requested date range fetched directly |
| Forecasts | Quarter-ahead country projections and public commodity/currency/crypto/index/bond forecasts |
| Holidays and earnings | Public/bank holidays and corporate EPS/revenue release calendars |
| News and search | Latest economic/market news stream and a keyword resolver for indicators and instruments |
| History and bonds | Indicator and market time series plus 15 government-bond maturities per country |

## How it works

The actor reads the same public, no-login pages a browser sees: market tables, country pages, rankings, calendars, forecasts, holiday lists, earnings tables, rating pages, bond pages, news and search results. Historical series come from the public chart data behind each indicator or market page. It expands a requested date range into short windows so the calendar does not stop at the page's display ceiling, removes duplicate records, and stops at `maxItems` or when the source runs out. When several countries, indicators or symbols are requested, records are emitted round-robin so one large target cannot consume the whole cap; history points are newest-first. No account, API key, cookie or proxy setup is required.

## Output

Every record carries `recordType` and `scrapedAt`. Fields below are the complete contract; a field is omitted when the source does not provide it.

### `market`
`name`, `marketType`, `category`, `maturity` (bonds), `unit`, `symbol`, `last`, `change`, `changePercent`, `weekly`, `monthly`, `ytd`, `yoy`, `marketCap`, `referenceDate`, `country`, `sourceUrl`.

### `countryIndicator`
`indicatorName`, `country`, `category`, `last`, `previous`, `highest`, `lowest`, `unit`, `referenceDate`, `sourceUrl`.

### `indicatorRanking`
`indicator`, `indicatorName`, `country`, `last`, `previous`, `referenceDate`, `unit`, `nextRelease`, `sourceUrl`. For `indicator=rating`: `spRating`, `dbrsRating`, `teScore`, `ratingDescription` (Moody's is not published publicly and is therefore omitted).

### `calendarEvent`
`eventId`, `event`, `country`, `countryCode`, `category`, `symbol`, `importance`, `date`, `time`, `referencePeriod`, `actual`, `previous`, `consensus`, `forecast`, `actualValue`, `previousValue`, `consensusValue`, `forecastValue`, `previousRevisedFrom`, `surprise`, `surpriseDirection`, `sourceUrl`.

### `countryForecast`
`indicatorName`, `country`, `category`, `unit`, `actual`, `forecasts[]` (`period`, `value`), `sourceUrl`.

### `holiday`
`date`, `country`, `countryCode`, `holidayName`, `sourceUrl`.

### `earnings`
`company`, `symbol`, `country`, `countryCode`, `date`, `epsActual`, `epsConsensus`, `epsPrevious`, `revenueActual`, `revenueConsensus`, `revenuePrevious`, `marketCap`, `fiscalPeriod`, `time`, `importance`, `sourceUrl`.

### `indicatorHistoryPoint`
`country`, `indicatorName`, `indicatorSlug`, `seriesSymbol`, `category`, `unit`, `frequency`, `source`, `date`, `timestamp`, `value`, `sourceUrl`.

### `marketHistoryPoint`
`symbol`, `name`, `date`, `timestamp`, `value`, `change`, `changePercent`, `interval`, `sourceUrl`.

### `marketForecast`
`name`, `assetClass`, `group`, `price`, `signal`, `forecasts[]` (`period`, `value`), `sourceUrl`.

### `news`
`newsId`, `title`, `description`, `url`, `author`, `country`, `category`, `importance`, `publishedAt`, `expiresAt`, `image`, `sourceUrl`.

### `searchResult`
`title`, `url`, `resultType`, `group`, `category`, `country`, `frequency`, `unit`, `sourceUrl`.

### `creditRatingEvent`
`country`, `agency`, `rating`, `outlook`, `date`, `sourceUrl`.

### `bondYield`
`country`, `name`, `maturity`, `yield`, `dailyChange`, `monthlyChange`, `yearlyChange`, `date`, `sourceUrl`.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `markets` | One of the 14 modes listed above |
| `marketType` | string | `commodities` | mode=markets: `commodities` / `currencies` / `stocks` / `bonds` / `crypto` / `shares` |
| `category` | string | – | mode=markets: category or region filter |
| `countries` | array | `[]` | Countries for countryIndicators, countryForecast, creditRatingHistory and bondYields (required there); optional for calendar, holidays and earningsCalendar |
| `indicatorCategory` | string | – | mode=countryIndicators / countryForecast: one macro category |
| `rankingIndicators` | array | `["interest-rate"]` | mode=indicatorRanking: one or more indicator slugs |
| `indicatorHistoryUrls` | array | `[]` | mode=indicatorHistory: TE indicator URLs or `country/indicator-slug` |
| `marketSymbols` | array | `[]` | mode=marketHistory: symbols such as `CL1:COM`, `AAPL:US` or TE market URLs |
| `forecastAssetClass` | string | `commodity` | mode=marketForecast: `commodity` / `currency` / `crypto` / `stockMarket` / `governmentBond10Y` |
| `searchQuery` | string | – | mode=search: keyword to resolve, e.g. `gold` |
| `historySpan` | string | `5y` | history modes: `1m` … `10y`, `max` |
| `marketInterval` | string | `auto` | mode=marketHistory: `1m` … `1M` or `auto` |
| `searchKeyword` | string | – | Case-insensitive filter on the record name/title |
| `minChangePercent` | number | – | mode=markets: minimum daily % change |
| `maxChangePercent` | number | – | mode=markets: maximum daily % change |
| `dateFrom` | string | – | ISO date lower bound for date-aware modes |
| `dateTo` | string | – | ISO date upper bound for date-aware modes |
| `importance` | string | – | calendar / earnings / news: `low` / `medium` / `high` |
| `maxItems` | integer | `100` | Hard cap on emitted records (1–10,000); shared round-robin across requested countries/indicators/symbols |

### Example: live energy commodities

```json
{
  "mode": "markets",
  "marketType": "commodities",
  "category": "Energy",
  "maxItems": 50
}
```

### Example: prices indicators for three countries

```json
{
  "mode": "countryIndicators",
  "countries": ["united-states", "germany", "japan"],
  "indicatorCategory": "prices",
  "maxItems": 100
}
```

### Example: US CPI history, full available series

```json
{
  "mode": "indicatorHistory",
  "indicatorHistoryUrls": ["united-states/inflation-cpi"],
  "historySpan": "max",
  "maxItems": 2000
}
```

### Example: high-impact calendar for one week

```json
{
  "mode": "calendar",
  "dateFrom": "2026-11-02",
  "dateTo": "2026-11-08",
  "importance": "high",
  "maxItems": 100
}
```

### Example: intraday crude-oil prices

```json
{
  "mode": "marketHistory",
  "marketSymbols": ["CL1:COM"],
  "historySpan": "1d",
  "marketInterval": "5m",
  "maxItems": 500
}
```

## Use cases

- **Trading and macro desks** — pull quotes, curves and calendar events into a research pipeline
- **Cross-country comparison** — rank every country by rates, inflation, debt or credit rating
- **Historical research** — export multi-decade indicator series without an API subscription
- **Fintech dashboards** — feed live markets, forecasts and bond curves into a client product
- **Event monitoring** — watch high-impact releases and news for market-moving surprises
- **Journalism and academia** — bulk-export macro data with source links and timestamps

## Usage cost

Apify bills this actor per result (one dataset item = one record). Market snapshots and rankings are small runs; history modes can emit thousands of points, so set `maxItems` to the number of points you actually need. The daily prefill is intentionally the smallest run with output.

## FAQ

**Is an API key or login required?** No. The actor reads public Trading Economics pages and chart data, and works with no account, cookie or proxy setup.

**Is this affiliated with Trading Economics?** No. It is an independent third-party actor that reads the same public pages a browser would see.

**How fresh is the data?** Market quotes are delayed intraday values, not tick-level real-time data. Indicators and calendar events update when Trading Economics republishes a release.

**Why is `moodysRating` missing from credit ratings?** Moody's ratings are not published on the public rating pages. The actor never fabricates them; S&P, DBRS and the Trading Economics score are returned instead.

**What is `teScore` and `ratingDescription`?** `teScore` is Trading Economics' 0–100 sovereign score. `ratingDescription` maps that score to the published grade band, such as Prime, High grade or Non-investment grade.

**How does the calendar surprise work?** `surprise` is actual minus consensus (or the published forecast when consensus is absent), emitted only when both values use the same currency, scale and unit. `surpriseDirection` is `above`, `below` or `inline`.

**Why do some calendar events lack values?** A future release has no actual yet, and some agencies publish only a previous value. Missing fields are omitted rather than filled with zero.

**Why do index and bond market forecasts show only a few rows?** Trading Economics publishes the Major group publicly; further regional index and bond forecast tables are subscription-only. Commodity, currency and crypto forecasts are public in full.

**Can I request more than one country or indicator?** Yes. Use the `countries`, `rankingIndicators` and history/market arrays. The actor fetches each target and emits round-robin, so a cap of 30 with three indicators returns about ten rows per indicator instead of thirty from the first one. Raise `maxItems` if you need every row from every target.

**Why does a history run stop before my `maxItems`?** The source itself has no more observations for that series or span, or the remaining targets produced nothing. The actor reports this rather than padding the dataset. History points are emitted newest-first so a capped run keeps the most recent observations.

## Notes

- Dates are ISO `YYYY-MM-DD`; `timestamp` is epoch seconds when the source provides one.
- Percent fields are whole percent values (for example `2.5` means 2.5%).
- `revenueActual`, `marketCap` and similar scaled figures keep their source suffix (`B`, `M`, `T`) because the scale can vary by company.
- Records are deduplicated by their natural key, so paginated calendar or news runs do not repeat rows.
- Multi-target runs make one request per requested country/indicator/symbol; `maxItems` is shared round-robin, so each target is represented before any gets a second row.
- Older saved tasks that still send `country` or `indicator` keep working; those values are mapped onto `countries` and `rankingIndicators` automatically.

## Limitations

- **Moody's ratings** are not in the public data and are never emitted.
- **Regional index and bond forecasts** beyond the Major group are paywalled by the source.
- **Calendar rendering** is capped by the source at roughly 2,000 rows per page; the actor splits long date ranges into short windows to stay under the cap.
- **News** is available 100 items per request and has no server-side country filter; the actor paginates and filters client-side.
- **Search** returns the site's fixed embedded hit set for a query rather than an open-ended index.
- **Market tables** have source-side row caps that vary by market type; use category runs for the deepest coverage.
- **Some symbols have no public chart history.** Certain futures contracts and less-covered instruments return an empty chart payload. The actor names the symbol in its status message and continues with the rest of the request.
- **Company financial statements** are not available on the public pages; only the earnings calendar fields are returned.
- **Small or low-coverage economies** may have no forecast, rating history or bond curve; those runs return the countries that do have data and leave the rest empty.
