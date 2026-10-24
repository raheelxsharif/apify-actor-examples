# Google Autocomplete Keyword Expansion Scraper

Turn a handful of seed keywords into hundreds of long-tail variants by running them through Google's own autocomplete endpoint — the same suggestions you'd see typing into the search box — expanded with question words, prepositions, comparison terms, or a full A–Z sweep.

## What this actor does

- Expands each seed keyword using one of six modes: exact match, Google's default related suggestions, question-word prefixes, preposition modifiers, comparison terms, or an A-to-Z letter sweep.
- Targets any Google market and language combination via country (`gl=`) and language (`hl=`) codes, so the same seed can be run per-region.
- Filters the resulting suggestions by minimum/maximum character length and an optional substring match (or its inverse, to exclude terms).
- Returns either one record per suggestion (`flat`) or one record per seed keyword with its suggestions nested inside (`tree`).
- Suggestion metadata Google doesn't return for a given query (such as `relevance` or `sub_types`) is left out of that record instead of being filled with a placeholder.

## Output per suggestion (flat mode)

- `recordType` — always `suggestion`
- `text` — the suggested search phrase
- `source_keyword` — the seed keyword this suggestion came from
- `expansion_query` — the exact query sent to Google Suggest to produce this batch (e.g. `python tutorial f`), when different from the seed
- `mode` — the expansion mode used
- `country`, `language` — the market and language codes used for the request
- `relevance` — Google's own relevance score for the suggestion, when returned
- `suggestion_type` — Google's suggestion type (e.g. `QUERY`), when returned
- `sub_types` — Google's numeric sub-type codes, when returned
- `scrapedAt` — ISO 8601 timestamp of when the record was scraped

## Output per seed keyword (tree mode)

- `recordType` — always `keyword_tree`
- `source_keyword`, `mode`, `country`, `language`
- `suggestion_count` — number of suggestions nested under this seed
- `suggestions[]` — each suggestion's `text`, `relevance`, and other per-suggestion fields, minus the seed and timestamp
- `scrapedAt`

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `keywords` | array | `["python tutorial"]` | Seed keywords to expand (required) |
| `mode` | string | `all` | `exact`, `all`, `questions`, `prepositions`, `comparisons`, or `alphabet` |
| `country` | string | `US` | Two-letter Google market code (`gl=`), or `ALL` |
| `language` | string | `en` | Two-letter Google language code (`hl=`) |
| `maxItemsPerKeyword` | integer | `200` | Cap on suggestions returned per seed keyword (1–5000) |
| `minLength` | integer | – | Drop suggestions shorter than this many characters |
| `maxLength` | integer | – | Drop suggestions longer than this many characters |
| `containsKeyword` | string | – | Keep only suggestions containing this substring; prefix with `!` to exclude matches instead |
| `outputFormat` | string | `flat` | `flat` (one record per suggestion) or `tree` (one record per seed) |

### Example: question-style suggestions for a topic

```json
{
  "keywords": ["intermittent fasting"],
  "mode": "questions",
  "country": "US",
  "language": "en",
  "maxItemsPerKeyword": 100
}
```

### Example: A-to-Z long-tail sweep, excluding a term

```json
{
  "keywords": ["python tutorial"],
  "mode": "alphabet",
  "containsKeyword": "!free",
  "maxItemsPerKeyword": 200
}
```

### Example: nested tree output across two markets

```json
{
  "keywords": ["running shoes"],
  "mode": "comparisons",
  "country": "GB",
  "language": "en",
  "outputFormat": "tree"
}
```

## Use cases

- **SEO content planning** — surface the long-tail questions and phrases people actually type
- **PPC keyword discovery** — expand a seed into broad-match and negative-match candidates
- **Competitor and brand research** — pair a brand name with `comparisons`/`prepositions` modes to see how people compare it
- **International SEO** — run the same seed across multiple `country`/`language` pairs to compare regional phrasing
- **Content gap analysis** — use `questions` mode to map "people also ask"-style topics for an article outline

## FAQ

**Does this need a login or API key?**
No — Google Suggest is a public, unauthenticated endpoint.

**How many suggestions can I get per seed?**
Google typically returns up to around 10 per individual query. `alphabet` mode issues 26 queries (one per letter) for up to ~260 raw suggestions per seed before filtering and deduplication; `questions` and `prepositions` modes scale with how many modifier words they use.

**What's the difference between `country` and `language`?**
`country` (`gl=`) shifts which suggestions Google considers geographically relevant — the same seed can return different results in different markets. `language` (`hl=`) controls the language Google serves suggestions in.

**Can I get search volume numbers?**
No. True search volume requires Google's paid Keyword Planner. The `relevance` score this actor returns is Google Suggest's own popularity signal and is useful as a relative proxy, not an exact volume.

**What is `expansion_query`?**
The literal query the actor sent to Google Suggest to produce a given batch of results (for example `python tutorial f` under alphabet mode). Google can still return suggestions that don't start with that exact text.

**What happens if my filters remove every suggestion for a seed?**
That seed is skipped and the run finishes normally with a status message noting which filters were active, rather than returning fabricated rows.

**Is this affiliated with Google?**
No. This is an independent, third-party actor that reads Google's public autocomplete endpoint and is not operated by or affiliated with Google.
