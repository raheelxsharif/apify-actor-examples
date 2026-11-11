# Decrypt Crypto News Finder by Category & Keyword

Decrypt publishes crypto, Web3, AI, policy and market news across a deep archive and five languages. This actor searches that archive by keyword, browses 47 categories, tags, 121 coins or authors, looks up specific articles by ID or URL, and returns full article records with HTML and plain text, TL;DR bullets, editors, author profiles, images, taxonomy and reading metrics. No account, cookies or proxy setup is required.

## What this actor does

- **Seven modes:** keyword search, browse by category, lookup by ID/URL, latest articles, browse by tag, browse by coin and browse by author
- **Deep archive access** with server-side date ranges, newest/oldest/relevance sorting and up to 10,000 articles per run
- **Full article data:** HTML body, plain-text body, TL;DR, editors, author bios and social profiles, tags, coins, collections, distinctions and images
- **Five locales:** English, Spanish, French, Arabic and Simplified Chinese, with the correct localized article URL
- **Batch lookups:** paste article IDs or decrypt.co URLs, including localized and `/resources/` links
- **Scheduled monitoring:** `onlyNew` skips articles returned by earlier runs
- **Empty fields are omitted** from every record instead of being returned as null

## Output per article

- `id` — Decrypt article ID
- `title`, `slug`, `excerpt`, `blurb` — headline and summaries
- `locale`, `status`, `publishedAt`, `modifiedAt` — language, publication status and ISO timestamps
- `content` — full article body as HTML
- `contentText` — plain-text article body for search, NLP and summaries
- `wordCount`, `readingTimeMinutes` — body word count and reading time in minutes
- `readingTime` — original reading-time label from the source
- `tldrTitle`, `tldrContent`, `tldrText` — "In brief" heading, HTML bullets and plain-text bullets
- `authors[]`, `authorSlugs[]` — byline names and slugs
- `authorDetails[]` — `name`, `slug`, `title`, `bio`, `location`, `twitter`, `linkedin`, `avatarUrl`, `roles`
- `editors[]`, `editorSlugs[]` — editor names and slugs
- `category`, `categorySlug`, `categoryParent`, `categoryParentSlug` — leaf category and its parent section
- `tags[]`, `tagSlugs[]` — topic names and filterable slugs
- `coins[]`, `coinSlugs[]`, `coinSymbols[]` — mentioned coins, slugs and ticker symbols
- `collections[]`, `collectionSlugs[]` — editorial collections
- `distinctions[]`, `distinctionSlugs[]` — editorial distinctions
- `celebrities[]`, `celebritySlugs[]` — people linked to the article
- `featuredImageUrl`, `featuredImageAlt`, `featuredImageWidth`, `featuredImageHeight` — featured image
- `featuredVideoUrl` — featured video URL when the article has one
- `metaDescription`, `socialImageUrl` — SEO description and social-sharing image
- `sourceUrl`, `canonicalUrl` — localized article URL
- `recordType` — always `"article"`
- `scrapedAt` — timestamp of when the record was scraped

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string (enum, required) | `search` | `search`, `browseByCategory`, `byId`, `trending`, `byTag`, `byCoin` or `byAuthor` |
| `searchQuery` | string | `""` | Keyword matched against titles, excerpts and bodies (mode `search`) |
| `category` | string (enum) | `""` | One of 47 category slugs; empty means any category |
| `categories` | array | `[]` | Several category slugs at once |
| `tagSlugs` | array | `[]` | Tag slugs, e.g. `bitcoin`, `regulation` |
| `coinSlugs` | array | `[]` | Coin slugs, names or tickers, e.g. `bitcoin`, `ethereum`, `SOL` |
| `authorSlugs` | array | `[]` | Author slugs, e.g. `jose` |
| `articleIds` | array | `[]` | IDs or decrypt.co URLs (mode `byId`) |
| `locale` | string (enum) | `""` | `en`, `es`, `fr`, `ar`, `zh-hans`; empty means all languages |
| `dateRangeFrom` | string | – | Only articles published on or after this date (`YYYY-MM-DD`) |
| `dateRangeTo` | string | – | Only articles published on or before this date (`YYYY-MM-DD`) |
| `sortBy` | string (enum) | `newest` | `newest`, `oldest` or `relevance` |
| `includeContent` | boolean | `true` | Include HTML/plain-text bodies, TL;DR content and word count |
| `onlyNew` | boolean | `false` | Skip article IDs returned by earlier runs |
| `maxItems` | integer | `10` | Maximum articles to return (1–10,000) |
| `proxyConfiguration` | object | off | Optional override; the actor works without a proxy |

### Example: keyword search with relevance and a date window

```json
{
  "mode": "search",
  "searchQuery": "etf inflows",
  "sortBy": "relevance",
  "dateRangeFrom": "2026-01-01",
  "dateRangeTo": "2026-03-31",
  "locale": "en",
  "maxItems": 200
}
```

### Example: browse a category in Spanish without article bodies

```json
{
  "mode": "browseByCategory",
  "category": "markets",
  "locale": "es",
  "includeContent": false,
  "maxItems": 100
}
```

### Example: look up a mixed batch of IDs and URLs

```json
{
  "mode": "byId",
  "articleIds": [
    "382316",
    "https://decrypt.co/373492/bitcoin-64000-inflacion-ipc-junio-fed-tasas-interes",
    "https://decrypt.co/resources/what-is-eip-7702-the-ethereum-upgrade-behind-the-6-4b-binance-hack"
  ]
}
```

### Example: monitor one coin for new stories

```json
{
  "mode": "byCoin",
  "coinSlugs": ["solana"],
  "dateRangeFrom": "2026-10-01",
  "onlyNew": true,
  "sortBy": "newest",
  "maxItems": 50
}
```

## Use cases

- **Crypto news monitoring** — track coverage of a coin, project, company or person across the archive
- **NLP and sentiment research** — feed clean `contentText` and `tldrText` into your own models
- **Media intelligence** — compare category, tag and author coverage over specific date windows
- **Content aggregation** — power a newsletter, digest or research dashboard with full articles
- **Editorial archiving** — build a localized archive across English, Spanish, French, Arabic and Chinese
- **Scheduled alerts** — use `onlyNew` to pick up only stories published since the last run

## FAQ

**Is this affiliated with Decrypt?**
No. This is an independent, third-party actor that reads publicly available Decrypt article pages and feeds. It is not operated by or affiliated with Decrypt.

**Do I need an account, API key or cookie?**
No. The actor runs without credentials and without proxy configuration.

**Does it return full article text or just a summary?**
Both. `content` is the complete HTML body and `contentText` is a plain-text version. `excerpt`, `blurb` and the TL;DR fields are also included when the article has them.

**Why are some fields missing from a record?**
Not every article has a coin, collection, distinction, editor, TL;DR or video. Fields with no value are omitted rather than returned empty or null. Decrypt's coin filter can also match stories whose article record has no explicit coin relation, so a `byCoin` run may include rows without `coins` or `coinSlugs`; the actor reports the article honestly instead of inventing a coin.

**What do `tagSlugs`, `coinSlugs` and `authorSlugs` expect?**
Tag and author filters use the slugs returned in each record, not display names. Coin filters accept slugs plus common names and ticker symbols, which are mapped to slugs automatically.

**How fresh is the data, and is `trending` a real trend signal?**
The archive is read live at run time. `trending` is the legacy name for the newest-first mode; it returns the latest published articles, not a proprietary popularity score.

**How many articles can one run return?**
Up to 10,000 with `maxItems`. The source exposes a very large archive; for very broad queries, split the work by date range or category to keep each run focused.

**Do localized articles have working URLs?**
Yes. `sourceUrl` and `canonicalUrl` use the article's own locale path, including `/es/`, `/fr/`, `/ar/` and `/zh-hans/`.

**Are sentiment scores or AI analysis included?**
No. Those values are not published by the source, and the actor does not fabricate them. Use `contentText` or `tldrText` with your own analysis pipeline.

**Can I run this on a schedule?**
Yes. Set `onlyNew` to `true` and the actor will skip article IDs returned by earlier runs, so each scheduled run returns only unseen stories.
