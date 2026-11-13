# Crypto Market Sentiment Index Tracker

Scrape the **Crypto Fear & Greed Index** — the market-sentiment gauge that scores the whole crypto market from **0 (Extreme Fear)** to **100 (Extreme Greed)**. Get the live reading, the last N daily values, or any historical date range, each with a numeric score, a sentiment classification, a unix timestamp, and clean ISO dates. Powered by the public `alternative.me` API. **No API key, no login, no proxy.**

## What this actor does

- **Three modes:** `latest`, `historical`, `dateRange`
- **Live sentiment:** the current index value plus seconds until the next update
- **Full history:** every daily reading since the index began (2018+)
- **Date-range slicing:** pull just the readings between two dates
- **Classification filter:** keep only `Extreme Fear`, `Fear`, `Neutral`, `Greed`, or `Extreme Greed` readings
- **Clean, typed output:** strings from the API are converted to real numbers and ISO dates
- **Empty fields are omitted** — every field you see is populated

## Output per reading

| Field | Type | Description |
|---|---|---|
| `value` | int | Index score, `0`–`100` |
| `valueClassification` | string | One of `Extreme Fear`, `Fear`, `Neutral`, `Greed`, `Extreme Greed` |
| `timestamp` | int | Unix time (seconds, UTC) of the reading |
| `date` | string | `YYYY-MM-DD` (UTC) derived from the timestamp |
| `datetimeUtc` | string | Full ISO 8601 UTC datetime |
| `timeUntilUpdate` | int | Seconds until the next update — **latest reading only** |
| `recordType` | string | Always `fearGreedIndex` |
| `scrapedAt` | string | ISO 8601 UTC time the record was scraped |

## Output: error record (mode = `dateRange` only, bad input)

| Field | Type | Description |
|---|---|---|
| `recordType` | string | Always `error` |
| `error` | string | Short error code, e.g. `invalid_date`, `invalid_date_range`, `missing_input` |
| `message` | string | Human-readable explanation of the problem |
| `input` | string | The offending input value |
| `sourceUrl` | string | The `alternative.me` endpoint the actor was about to call |
| `scrapedAt` | string | ISO 8601 UTC time the record was generated |

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `latest` | `latest` / `historical` / `dateRange` |
| `limit` | int | `30` | Number of most-recent daily readings (mode=historical), `1`–`5000` |
| `startDate` | string | – | Inclusive range start, `YYYY-MM-DD` (mode=dateRange) |
| `endDate` | string | – | Inclusive range end, `YYYY-MM-DD` (mode=dateRange) |
| `classificationFilter` | string | – | Keep only one sentiment band (mode=historical / dateRange) |

### Example: current reading

```json
{
  "mode": "latest"
}
```

### Example: last 90 daily readings

```json
{
  "mode": "historical",
  "limit": 90
}
```

### Example: only the "Extreme Fear" days in a range

```json
{
  "mode": "dateRange",
  "startDate": "2024-01-01",
  "endDate": "2024-12-31",
  "classificationFilter": "Extreme Fear"
}
```

## Use cases

- **Trading signals** — feed sentiment into contrarian buy/sell strategies
- **Dashboards & bots** — display the live index in Telegram/Discord/Slack
- **Backtesting** — align historical sentiment against price data
- **Research** — study how fear and greed track market cycles
- **Alerts** — trigger a notification when the market flips into Extreme Fear or Extreme Greed
- **Content** — power newsletters and widgets with a fresh daily reading

## FAQ

**What is the Crypto Fear & Greed Index?**  A daily 0–100 score of overall crypto-market sentiment, published by alternative.me. Low values mean investors are fearful (often seen as a buying opportunity); high values mean the market may be greedy and due for a correction.

**How are the classifications defined?**  Each score maps to a band: `0–25` Extreme Fear, `26–46` Fear, `47–54` Neutral, `55–75` Greed, `76–100` Extreme Greed.

**How often does it update?**  Once per day. The `latest` reading also includes `timeUntilUpdate`, the number of seconds until the next refresh.

**How far back does the history go?**  To the index's launch in early 2018. `historical` and `dateRange` can reach the entire archive.

**Do I need an API key or proxy?**  No. The data source is a free public API — this actor uses no authentication, cookies, or proxy.

**What happens with a bad date range?**  If a date is unparseable, or `startDate` is after `endDate`, the actor emits a single typed error record explaining the problem instead of failing silently.

**Why do some fields not appear?**  The actor omits any empty field. For example, `timeUntilUpdate` only appears on the current reading, never on historical rows.

**Is the index the same as the stock-market Fear & Greed Index?**  No — this is the crypto-specific index from alternative.me, built from volatility, momentum, volume, social media, dominance, and trends across the crypto market.
