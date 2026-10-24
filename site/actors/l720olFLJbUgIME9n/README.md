# Google Autocomplete Keyword Expansion Scraper

Turn a short seed keyword into the long-tail phrases people actually type into Google — then keep the provenance that makes the list usable: which query surfaced each phrase, where it ranked, which market and language it came from, and Google's own relevance score.

Suggestions come straight from Google's public autocomplete endpoint. No login, no API key, no browser, no cookies.

## What it does

- Runs one of eight expansion modes per seed, from a single exact query to a full sweep that combines alphabet, digits, questions, modifiers, comparisons and related variants.
- Adds letters and numbers **before and after** the seed (`coffee a`, `coffee 1`, `a coffee`, `1 coffee`), not just the usual A–Z suffix.
- Uses native question and modifier vocabularies for 19 languages, so Spanish and Japanese seeds are expanded with Spanish and Japanese words instead of English ones.
- Can query Google web, YouTube and Shopping in one run.
- Optionally re-queries the first 15 discoveries one or two levels deeper.
- Targets 134 verified Google markets and 78 verified interface languages.
- Caps the run with a global record limit and a per-seed suggestion limit. The per-seed cap limits collected rows rather than aborting the A–Z sweep, so a full alphabet sweep is never silently cut short.
- Keeps the old flat and tree record shapes, so existing tasks and integrations keep working.

## Modes

| Mode | What it sends | Typical unique phrases per seed |
|---|---|---:|
| `exact` | The seed only. | 10–15 |
| `alphabet` | `seed a` … `seed z`, `seed 0` … `seed 9`, and the same tokens before the seed. | ~1,000 |
| `questions` | Native question words before and after the seed. | ~300–450 |
| `prepositions` | Native prepositions after the seed. | ~50–150 |
| `comparisons` | `vs`, `or`, `versus` after the seed and `vs seed`. | ~15–50 |
| `modifiers` | Prepositions plus buyer-intent words before and after the seed. | ~300–470 |
| `related` | Trailing-space, wildcard, near-me, review and price variants. | ~30–70 |
| `all` | The deduplicated union of every strategy. | ~1,100–2,000 |

Counts are for mainstream 1–3 word seeds. Obscure or long seeds return far fewer suggestions because Google completes prefixes, not topics.

## Output

One flat row per suggestion (default):

| Field | Meaning |
|---|---|
| `keyword` / `text` | The suggested phrase. `text` is the legacy alias. |
| `seed` / `source_keyword` | The seed this phrase came from. `source_keyword` is the legacy alias. |
| `type` | Which expansion produced it: `suggestion`, `alphabet`, `question`, `modifier`, `preposition`, `comparison`, `related` or `recursive`. |
| `position` | Its 1-based rank in Google's response to that query. |
| `expansion_query` | The exact query sent to Google, when different from the seed. |
| `locale`, `country`, `language` | The market and interface language used. |
| `source` | `web`, `youtube` or `shopping`. |
| `sourceUrl` | The exact Suggest request URL, for traceability. |
| `relevance` | Google's own ranking score for the phrase, when the chrome dialect returns it. |
| `suggestion_type`, `sub_types` | Google's raw suggestion tag and numeric sub-type codes. |
| `verbatim_relevance` | Google's query-level verbatim score. |
| `depth`, `parent_keyword` | Recursion level and the query that produced a deeper suggestion. |
| `is_question`, `word_count`, `intent` | Deterministic local tags. `intent` is added when a language vocabulary matches. |
| `mode`, `recordType`, `scrapedAt` | Run metadata and the legacy record marker. |

`tree` mode emits one record per seed and source with a nested `suggestions[]` array and a `type_counts` summary. Its legacy keys (`source_keyword`, `mode`, `country`, `language`, `suggestion_count`, `suggestions`, `scrapedAt`) are unchanged.

## Input

| Field | Default | Notes |
|---|---|---|
| `keywords` | `["python tutorial"]` | Required. One or more seeds; 1–3 words work best. |
| `mode` | `all` | One of the eight modes above. |
| `country` | `US` | 134 verified Google markets, plus a legacy `ALL` value that maps to `US`. |
| `language` | `en` | 78 verified Google languages. |
| `source` | `web` | `web`, `youtube`, `shopping` or `all`. |
| `maxItems` | `5000` | Global record cap. Flat counts suggestions; tree counts seed/source records. |
| `maxSuggestionsPerSeed` | – | Cap on collected suggestions per seed; the depth-0 query plan still runs. |
| `minLength`, `maxLength` | unset | Drop suggestions outside the character range. |
| `containsKeyword` | unset | Keep suggestions containing a substring; prefix with `!` to exclude. |
| `outputFormat` | `flat` | `flat` or legacy `tree`. |
| `expansionDepth` | `0` | `0`, `1` or `2`: re-query the first 15 discoveries. |
| `dedupeAcrossSeeds` | `false` | Emit a phrase only once per run. |
| `includeIntent` | `true` | Add the deterministic intent tag. |

Old tasks that send `maxItemsPerKeyword` still work: it maps to `maxSuggestionsPerSeed`. `country: "ALL"` is mapped to `US` with a warning because Google has no `ALL` market.

The input prefill is one `python tutorial` seed in `exact` mode, capped at 3 records.

### Examples

Exact, cheapest run:

```json
{"keywords": ["coffee"], "mode": "exact", "maxItems": 3}
```

Full English sweep with one extra recursion level:

```json
{"keywords": ["running shoes"], "mode": "all", "country": "US", "language": "en", "expansionDepth": 1, "maxItems": 5000}
```

Spanish market with native question words on the web and YouTube:

```json
{"keywords": ["café"], "mode": "questions", "country": "ES", "language": "es", "source": "all"}
```

Legacy nested output:

```json
{"keywords": ["coffee"], "mode": "all", "outputFormat": "tree", "maxSuggestionsPerKeyword": 100}
```

## How it works

Each mode turns the seed into a fixed, deduplicated list of Suggest queries. The actor sends them with a browser-like client, reads the JSON array Google returns, and normalises four response dialects (chrome, firefox, safari, and the JSONP variants). It detects a block from the status code, an HTML content type, a `/sorry` URL or the "unusual traffic" page, and switches from a direct connection to its included Apify datacenter proxy. If the automatic proxy cannot be created, it retries with the lighter firefox dialect. There is no proxy setting to configure, no login and no API key.

## FAQ

**Do I need a proxy or a Google account?**
No. The actor starts with a direct connection and uses its included Apify proxy only after Google blocks the direct one. There is nothing to configure and no credential to supply.

**How many suggestions will one seed return?**
It depends on the seed and mode. A mainstream 1–3 word seed with `all` typically returns 1,100–2,000 unique phrases. A long or obscure seed can return fewer than 100 because autocomplete only completes prefixes.

**Why are there no search-volume numbers?**
Google autocomplete does not expose search volume. `relevance` is Google's internal ranking score for the phrase, and it is useful for ordering, not for estimating monthly searches. Use a separate keyword-volume source if you need exact demand.

**What is the difference between country and language?**
`country` (`gl`) changes which suggestions Google considers locally relevant. `language` (`hl`) changes the interface language and the native question/modifier words the actor uses. Setting both gives the most authentic local list.

**Can I get the same phrase from two seeds?**
By default yes, because each seed stays complete. Turn on `dedupeAcrossSeeds` to emit each phrase once per run.

**What does recursion do?**
`expansionDepth` 1 re-queries the first 15 suggestions, and depth 2 re-queries the first 15 results of that pass. It costs about 15 extra requests per level and can add hundreds of phrases, especially with `exact`.

**Why is a phrase I know missing?**
Google filters some commercial, medical and political suggestions, and it varies results by market and time. The actor reports what Google returns; it does not invent missing phrases.

**Does the actor work with non-Latin scripts?**
Yes. It preserves the original script and corrects the response charset, including Japanese, Korean, Chinese, Russian, Arabic, Hebrew, Greek, Polish, Turkish, Thai, Vietnamese and the Indic languages.

**Is this affiliated with Google?**
No. It reads Google's public autocomplete endpoint and is not operated by or affiliated with Google. You are responsible for how you use the output.

## Limits

- Google's autocomplete endpoint is undocumented and can change or throttle without notice.
- Heavy direct traffic can be blocked; the actor then switches to its included Apify datacenter proxy automatically. No residential proxy is used.
- Autocomplete completes prefixes, so very long seeds return little.
- There is no pagination: each query returns at most 15 suggestions (10 on the fallback dialect). Coverage comes from more queries, not more pages.
- Google may omit suggestions for sensitive or highly commercial queries.
- No search volume, CPC, difficulty, People Also Ask or entity metadata is fabricated.
