# Ubersuggest Keyword Suggestions by Country & Language

Pull **Ubersuggest**'s keyword-idea list for a seed keyword in the market and language you choose. Suggestions carry search volume, CPC, competition, SEO and paid difficulty, and search intent where Ubersuggest exposes them; the source unlocks full metrics for the first ~19 suggestions in each response. An optional overview adds the seed's own metrics. **231 countries plus worldwide, 122 languages, no login and no cookies.**

## What this actor does

- **Expand one or many seeds.** Give it `coffee`, `running shoes`, or `project management software`; each seed returns its own suggestion list.
- **Choose the market.** 231 countries plus a worldwide option (232 choices). Country changes the suggestions and the metrics, not just a label.
- **Choose the language.** 122 languages. With auto-match on, picking Germany, Japan or Brazil switches to the market's main language automatically.
- **Get real keyword metrics.** Search volume, CPC in USD and local currency, competition, SEO difficulty, paid difficulty, search intent and the source's freshness date.
- **Add a seed overview.** Optionally fetch the seed's own volume, CPC, difficulty, intent, a 4-month volume trend, global volume breakdown, device share and click share.
- **Go deep or stay small.** Cursor pagination can return thousands of suggestions per seed; per-seed and whole-run caps keep runs predictable.
- **Choose the shape.** Keep the classic one-record-per-seed output with nested suggestions, or switch to one flat record per suggestion.
- Empty fields are omitted; the actor never writes `null`, empty strings or empty arrays.

## Output per seed (default)

Each record is one seed keyword:

- `type` — `keyword`
- `keyword` — the seed you supplied
- `country`, `language` — the market and language actually used
- `locationId` — Ubersuggest's location id (`"0000"` for worldwide)
- `suggestions[]` — related keywords, each with:
  - `type` — `keyword`
  - `keyword`
  - `country`, `language`
  - `searchVolume`, `cpc`, `cpcLocal`, `currency`
  - `competition`, `seoDifficulty`, `paidDifficulty`
  - `searchIntent[]` — `informational`, `navigational`, `commercial`, `transactional`
  - `dataUpdatedAt`
- Metrics appear only on the rows Ubersuggest unlocks (the first ~19 in each response); the rest carry the identity fields.
- `seedOverview` — present when seed metrics are enabled:
  - `searchVolume`, `cpc`, `cpcLocal`, `currency`
  - `competition`, `seoDifficulty`, `paidDifficulty`, `searchIntent[]`
  - `monthlyTrend` — 4 monthly volume points
  - `globalSearchVolume`, `globalVolumeBreakdown`
  - `desktopShare`, `mobileShare`, `clickRate`, `paidClickShare`, `organicClickShare`
  - `dataUpdatedAt`
- `updated_at`, `scrapedAt`

## Output per suggestion (flat mode)

Set `outputFormat` to `perSuggestion` to get one row per suggestion:

- `type` — `keywordSuggestion`
- `seedKeyword`, `keyword`, `country`, `language`, `locationId`
- `searchVolume`, `cpc`, `cpcLocal`, `currency`
- `competition`, `seoDifficulty`, `paidDifficulty`, `searchIntent[]`
- `dataUpdatedAt`, `scrapedAt`

When seed metrics are enabled, a matching `type: "keywordOverview"` record is added once per seed with the overview fields. This is intentional: flat mode returns the seed row plus one row per suggestion (three suggestions with seed metrics on = four rows). With seed metrics off, flat mode returns only suggestion rows.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `keywords` | Array | – | Seed keywords. Each seed returns its own suggestions. |
| `urls` | Array | – | Optional domains/URLs converted into keyword seeds. A path URL uses its last usable segment (`/blog/coffee-guide` → `coffee guide`); a homepage uses the host label (`make.com` → `make`). Not a domain keyword report. |
| `country` | Select | `us` | 231 countries or `worldwide` (232 options). Legacy `uk` is accepted and output as `gb`; legacy `ka` maps to `la`. |
| `language` | Select | `en` | 122 languages. |
| `autoMatchLanguageToCountry` | Boolean | true | Switch English to the market's main language when one exists. |
| `sortBy` | Select | `-searchVolume` | Volume, CPC, SEO/paid difficulty or alphabetical. |
| `maxSuggestionsPerSeed` | Integer | 100 | Per-seed cap, 1–1000. |
| `maxItems` | Integer | 1000 | Whole-run suggestion cap, 1–10000. |
| `includeSeedOverview` | Boolean | true | Add the seed's own metrics, trend, global and device shares. |
| `outputFormat` | Select | `perSeed` | Nested seed records or flat suggestion rows. |

Legacy tasks: `keyword` maps to the seed list; `uk: true` wins over `country` and is output as `gb`; legacy `ka` maps to `la`; old `include_*`/proxy keys are accepted but ignored. Flat mode includes one `keywordOverview` seed row when `includeSeedOverview` is on.

### Example: default US English

```json
{
  "keywords": ["coffee"],
  "country": "us",
  "language": "en"
}
```

### Example: German market with automatic language

```json
{
  "keywords": ["Kaffee"],
  "country": "de",
  "autoMatchLanguageToCountry": true
}
```

### Example: Japanese seed and a deeper list

```json
{
  "keywords": ["コーヒー"],
  "country": "jp",
  "language": "ja",
  "maxSuggestionsPerSeed": 300,
  "maxItems": 300
}
```

### Example: flat rows with a seed overview

```json
{
  "keywords": ["running shoes", "trail shoes"],
  "country": "gb",
  "sortBy": "-searchVolume",
  "maxSuggestionsPerSeed": 100,
  "maxItems": 200,
  "includeSeedOverview": true,
  "outputFormat": "perSuggestion"
}
```

## Use cases

- **Keyword research** — expand a seed into the phrasings Ubersuggest actually suggests for a market.
- **Content planning** — find question-led and long-tail ideas, then sort by volume or difficulty.
- **Paid search** — pull CPC and paid difficulty for ad keyword lists.
- **Multi-market SEO** — compare the same seed across countries and languages.
- **SEO dashboards** — export flat rows into a warehouse, sheet or BI tool.
- **Agency workflows** — batch many client seeds with a predictable per-seed and total cap.

## FAQ

### Where does the data come from?

Ubersuggest's public keyword data. The actor reads only keyword suggestions and their metrics; it does not need an Ubersuggest account, cookie or API key.

### Are the search volumes exact?

No. Ubersuggest search volumes are estimates, rounded to practical buckets and refreshed on its own schedule. They are useful for prioritisation and for comparing ideas in the same market; they are not Google Ads' exact figures. CPC is Ubersuggest's advertiser estimate, in USD (`cpc`) and the market's currency (`cpcLocal`).

### Why do some suggestions have no volume, CPC or difficulty?

Ubersuggest exposes full metrics for the first ~19 suggestions in each response. The remaining keywords are still returned with their identity fields, but without volume/CPC/difficulty. Ubersuggest's bulk metrics endpoint repeats the same core fields for up to 10 keywords per request; it adds no new per-suggestion fields, so the actor does not make a separate enrichment pass and never guesses missing values.

### How many suggestions can I get per seed?

The list is cursor-paginated and can run into the thousands for broad seeds. `maxSuggestionsPerSeed` and `maxItems` stop the run; the source can also run out earlier. The actor does not publish a total result count because the source's match count is not the number of returned rows.

### What is the seed overview?

It is the metrics for the seed itself, not for a suggestion. It includes volume, CPC, competition, difficulty, intent, a **4-month** volume trend, global volume with a breakdown, and desktop/mobile and click shares. It adds one extra request per seed.

### Which countries and languages are supported?

231 countries, 122 languages and `worldwide` (232 options). Russia and Belarus are not available from Ubersuggest's location data and are intentionally absent. If an old task still asks for one of them, the actor explains the limitation instead of returning wrong data.

### What does auto-match do?

When `language` is `en` and the chosen market has another main language, auto-match switches to it: Germany to German, Japan to Japanese, Brazil to Portuguese, China to Simplified Chinese. Set it to false to keep English suggestions for that market.

### Do I need a login, cookies or a proxy?

No. The actor reads public keyword data without an account, and it chooses its own working connection automatically. Just provide keywords and run it.

### Can I use a domain instead of a keyword?

`urls` is a legacy convenience. A path URL uses its last usable path segment (`https://example.com/blog/coffee-guide` becomes `coffee guide`); a homepage or a bare domain uses the host label (`make.com` becomes `make`). It does not analyse the domain's rankings, pages or backlinks. For real keyword research, use `keywords`.

### Is this affiliated with Ubersuggest?

No. This is an independent tool that reads publicly available keyword data.
