# YouTube Search Suggestions Scraper

Discover the phrases YouTube puts in its search box. Enter one or more seed queries and collect the platform's live autocomplete suggestions as clean, ranked JSON or CSV rows — with optional alphabet, question, modifier and related-search expansion for keyword research, video planning and audience research.

No YouTube account, API key, cookies or user-supplied proxy is needed. The actor reads YouTube's public autocomplete response and chooses a working connection automatically.

## What this actor does

- **Live YouTube suggestions:** each exact seed query returns up to 14 suggestions, the same list the signed-out YouTube search box shows.
- **Eight expansion modes:** `exact`, `alphabet` (suffix and prefix, letters and digits), `questions`, `modifiers`, `prepositions`, `comparisons`, `related`, and `all` (the union).
- **Optional recursion:** re-query the best suggestions to depth 1 or 2 for deeper long-tail coverage.
- **Locale targeting:** 134 countries and 78 languages, with localized question and modifier words for 19 languages.
- **Channel/topic cards:** when YouTube attaches a creator or topic card to a suggestion, its name, description or handle, image, channel ID and entity URL are included when supplied.
- **Filters and caps:** length, word count, question-only, contains-seed, exclude terms, per-seed and global row caps.
- **Per-seed deduplication:** repeated phrases are merged with a best rank and an appearance count. Optional cross-seed deduplication.
- **Empty fields are omitted:** records only contain values that are actually populated.

## Output per suggestion

| Field | Description |
|---|---|
| `recordType` | `suggestion` for a suggestion row, `status` when a seed produced nothing. |
| `id` | Stable 16-character ID for the source, seed, suggestion and locale. |
| `suggestion` | The exact phrase YouTube autocomplete returned. |
| `seed` | The seed query that produced this row. |
| `inputQuery` | Alias of `seed`, kept for older integrations. |
| `query` | The exact expansion query that produced the best rank. |
| `type` | `suggestion`, `alphabet`, `question`, `modifier`, `preposition`, `comparison`, `related` or `recursive`. |
| `rank` | Best 1-based position across every query that returned the phrase. |
| `appearances` | Number of expansion queries that returned the phrase. |
| `depth` | 0 for the base plan, 1-2 for recursion. |
| `parentQuery` | The query that led to a recursive row. |
| `language`, `market`, `locale` | Requested language, country and combined locale. |
| `source` | `youtube` or `youtubeMusic`. |
| `suggestionType` | `query`, `channel` or `topic`. |
| `subTypes` | YouTube's raw numeric suggestion codes, when present. |
| `entityId`, `entityName`, `entityDescription` | Channel/topic card identity, name and description or handle. |
| `entityImageUrl`, `entityUrl` | Card image and source-provided entity link, when present. |
| `channelId`, `channelHandle`, `channelUrl` | Channel card details, when present. |
| `wordCount`, `isQuestion`, `containsSeed` | Derived text signals. |
| `searchUrl` | Ready-to-open YouTube or YouTube Music search URL. |
| `scrapedAt` | ISO-8601 UTC timestamp. |

When a seed returns no rows, the actor emits one status row instead: `recordType` (`status`), `status`, `seed`, `query`, `language`, `market`, `source` and `suggestionCount` (`0`). Status values are `no_suggestions`, `suggestions_unavailable`, `invalid_query` and `blocked`. An unavailable YouTube Music response is reported as `suggestions_unavailable` for the seed.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `queries` | array | `["web scraping"]` | Seed keywords or phrases. 1-3 word seeds work best. |
| `mode` | string | `exact` | `all`, `exact`, `alphabet`, `questions`, `modifiers`, `prepositions`, `comparisons` or `related`. |
| `source` | string | `youtube` | `youtube`, `youtubeMusic` or `both`. |
| `language` | string | `en` | Language code used to localize suggestions. |
| `market` | string | `US` | Country or region code used to localize suggestions. |
| `maxItems` | integer | `5000` | Global cap on suggestion rows. |
| `maxSuggestionsPerSeed` | integer | `2000` | Rows kept per seed after the expansion plan. |
| `maxQueriesPerSeed` | integer | `300` | Autocomplete requests per seed and source. |
| `expansionDepth` | integer | `0` | `0` = selected mode only; `1`-`2` = re-query the best suggestions. |
| `dedupeAcrossSeeds` | boolean | `false` | Keep one row when a phrase appears for several seeds. Per-seed dedupe is always on. |
| `minLength` | integer | – | Drop shorter suggestions. |
| `maxLength` | integer | – | Drop longer suggestions. |
| `containsKeyword` | string | – | Keep suggestions containing this text; prefix with `!` to exclude it. |
| `excludeTerms` | array | `[]` | Drop suggestions containing any of these terms. |
| `minWordCount` | integer | `0` | Drop suggestions with fewer words. |
| `onlyQuestions` | boolean | `false` | Keep only question-shaped suggestions. |
| `mustContainSeed` | boolean | `false` | Keep only suggestions that contain the seed. |
| `includeEntityCards` | boolean | `true` | Keep channel/topic cards and their metadata. |
| `includeSearchUrl` | boolean | `true` | Add a ready-to-open search URL to every row. |

Old tasks that use `keywords`, `maxSuggestions`, `deduplicateSuggestions` or `country` keep working; those legacy keys are mapped to the new fields automatically.

## Examples

### Example: exact suggestions for one seed

```json
{
  "queries": ["web scraping"],
  "mode": "exact",
  "language": "en",
  "market": "US",
  "maxItems": 20
}
```

### Example: full keyword expansion

```json
{
  "queries": ["coffee", "minecraft"],
  "mode": "all",
  "expansionDepth": 1,
  "maxItems": 5000
}
```

### Example: Spanish questions with channel cards

```json
{
  "queries": ["café"],
  "mode": "questions",
  "language": "es",
  "market": "ES",
  "includeEntityCards": true
}
```

### Example: YouTube Music suggestions

```json
{
  "queries": ["lofi hip hop"],
  "source": "youtubeMusic",
  "mode": "exact",
  "maxItems": 20
}
```

### Example: long-tail ideas that mention the seed

```json
{
  "queries": ["running shoes"],
  "mode": "all",
  "mustContainSeed": true,
  "minWordCount": 3,
  "maxItems": 1000
}
```

## Use cases

- **YouTube SEO and content planning** — turn one topic into the phrases viewers actually type.
- **Keyword research** — compare suggestions across markets and languages.
- **Title and script ideas** — collect question and long-tail variants for a niche.
- **Creator discovery** — surface channel cards YouTube attaches to short brand seeds.
- **Trend monitoring** — schedule a run and diff `suggestion` or `rank` between snapshots.
- **YouTube Music research** — read the separate music-biased suggestion list for artists, genres and moods.

## FAQ

### Does this require a YouTube account, API key or cookies?

No. It reads YouTube's public search-box suggestions and works signed out.

### How many suggestions does one query return?

YouTube returns up to 14 suggestions per query. There is no pagination and no parameter that raises that ceiling.

### Why are some suggestions missing or different?

Autocomplete is personalized by language, country, current trends and the request location, and YouTube can change it at any time. The actor records the requested locale but cannot guarantee a specific list.

### Can I get search volume or competition metrics?

No. YouTube autocomplete does not publish search volume, CPC or difficulty. Those numbers come from separate paid keyword or search-results products.

### Why does a long seed return nothing?

Autocomplete completes prefixes, so four- and five-word seeds often return few or no suggestions. Use a 1-3 word seed, or use `alphabet` mode to expand around a long seed.

### What are channel and topic cards?

Some suggestions carry a creator or topic card with a name, handle, image, channel ID or entity link. The actor includes those fields when YouTube supplies them. Set `includeEntityCards` to `false` if you only want plain query suggestions.

### How does deduplication work?

Duplicate phrases inside one seed are merged into one row with the best `rank` and an `appearances` count. Cross-seed duplicates are kept by default so each seed's list stays complete; set `dedupeAcrossSeeds` to `true` for one row per phrase across the whole run.

### What is the YouTube Music source?

YouTube Music has its own music-biased suggestion list. It typically returns 6-10 suggestions per query and can return none for some seeds or locales; it is not a replacement for the main YouTube search box. Use `source: "youtubeMusic"` for music research or `both` to collect both lists.

### Does this support YouTube TV suggestions?

No. YouTube TV does not expose a separate public suggestion list; it uses the same search suggestions as YouTube.

### Do I need to set up a proxy?

No. The actor chooses a working connection to YouTube automatically. Just fill in the input and run it.

---

This is a third-party actor that reads publicly available YouTube autocomplete data. It is not affiliated with, endorsed by or sponsored by YouTube or Google.
