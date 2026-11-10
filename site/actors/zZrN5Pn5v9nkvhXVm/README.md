# Medium Article Discovery & Metadata Extractor

Medium is full of useful writing, but browsing it by hand makes it hard to compare coverage across topics, authors and publications. This actor turns five common discovery paths — tag browsing, author feeds, publication feeds, keyword search and direct article URLs — into one consistent article dataset you can sort, filter and join with your own tools.

Every record describes a single article: who wrote it, where it was published, when it went live, how long it takes to read, how much engagement it has, whether it sits behind Medium's member paywall, and which topics and tags it carries.

## Discovery modes

| Mode | What it collects | Best for |
|---|---|---|
| `byTag` | Articles tagged with a topic, newest first or by top stories from the week/month/year | Trend tracking and editorial research |
| `byUser` | An author's articles, including posts distributed to publications | Monitoring a writer's output |
| `byPublication` | Articles published under a publication | Following a publication's latest work |
| `search` | Articles matching a keyword query | Broad topic discovery |
| `byUrl` | Metadata for one or more article URLs | Enriching a list of links you already have |

Tag, author, publication and search modes walk the source's pagination until `maxItems` is reached or the source runs out, so a run can return thousands of records rather than a single page.

## Output fields

| Field | Description |
|---|---|
| `articleId` | Medium's short article identifier |
| `title` | Article headline |
| `subtitle` | Article subtitle or standfirst, when present |
| `previewText` | Subtitle or search-style preview text, capped at 300 characters |
| `authorName` | Author display name |
| `authorUsername` | Medium handle without `@` |
| `authorBio` | Author bio, when the profile has one |
| `authorImageUrl` | Author avatar URL |
| `authorUrl` | Author profile URL |
| `publicationName` | Publication name, when the article belongs to one |
| `publicationSlug` | Publication slug |
| `publicationDomain` | Custom publication domain, when one is used |
| `publicationUrl` | Publication home URL |
| `publishedDate` | First publish time, ISO-8601 UTC |
| `updatedDate` | Latest update time, when it differs from `publishedDate` |
| `tags` | Article tag names |
| `topics` | Medium topic names attached to the article |
| `readingTimeMinutes` | Estimated reading time in whole minutes |
| `wordCount` | Article word count |
| `language` | Detected language code |
| `clapCount` | Clap count reported by Medium |
| `responseCount` | Number of responses on the article |
| `isPaywalled` | `true` when the article is member-only |
| `isSeries` | Present and `true` when the article belongs to a series |
| `articleUrl` | Canonical article URL from Medium |
| `canonicalUrl` | Canonical URL when it differs from `articleUrl` |
| `imageUrl` | Article preview image URL |
| `recordType` | Always `article` |
| `siteName` | Always `Medium` |
| `scrapedAt` | When the record was collected, ISO-8601 UTC |

Empty fields are omitted rather than written as `null`. A field appears only when Medium exposes that value for the article.

## Input fields

| Field | Type | Default | Used by |
|---|---|---|---|
| `mode` | select | `byTag` | All modes |
| `tag` | select | `programming` | `byTag` |
| `customTag` | text | – | `byTag` (overrides `tag`) |
| `sortBy` | select | `new` | `byTag` |
| `username` | text | – | `byUser` |
| `publication` | text | – | `byPublication` |
| `query` | text | `artificial intelligence` | `search` |
| `articleUrl` | text | – | `byUrl` |
| `fromDate` | text | – | All modes |
| `containsKeyword` | text | – | All modes |
| `excludePaywalled` | boolean | `false` | All modes |
| `maxItems` | integer | `5` | All modes |

### Example: newest Python articles

```json
{
  "mode": "byTag",
  "tag": "python",
  "sortBy": "new",
  "maxItems": 25
}
```

### Example: an author's recent work

```json
{
  "mode": "byUser",
  "username": "towardsdatascience",
  "maxItems": 50
}
```

### Example: a publication's archive

```json
{
  "mode": "byPublication",
  "publication": "better-programming",
  "maxItems": 100
}
```

### Example: keyword search with a date cut-off

```json
{
  "mode": "search",
  "query": "large language models",
  "fromDate": "2025-01-01",
  "excludePaywalled": true,
  "maxItems": 40
}
```

### Example: enrich links you already have

```json
{
  "mode": "byUrl",
  "articleUrl": "https://medium.com/@author/example-article-abc123def456\nhttps://medium.com/p/abcdef123456",
  "maxItems": 10
}
```

## What you can do with it

- Build a topic monitoring sheet that shows which authors and publications are covering a subject each week.
- Compare top and newest stories for a tag to separate evergreen hits from fresh coverage.
- Filter out member-only posts before sending articles to a newsletter or internal digest.
- Enrich a list of saved links with author, publication, date, engagement and topic metadata.
- Track an author's or publication's output over time with scheduled runs.

## FAQ

**Do I need a Medium account or cookies?** No. The actor reads publicly available article metadata without logging in.

**Do I need to configure a proxy?** No. The actor picks a working connection automatically, so you never supply proxy settings, cookies or an API key.

**Can I get the full article text?** This actor focuses on discovery and metadata. It returns titles, previews, author and publication details and engagement metrics, not the article body.

**How far back can it go?** It follows the source's own pagination until `maxItems` is reached or the source runs out. Very large requests can span many pages, so raise `maxItems` deliberately.

**Why are some fields missing on some records?** Medium does not expose every field for every article. A personal post may have no publication, an older post may have no subtitle, and some articles have no tags. Missing values are omitted instead of filled with placeholders.

**How are paywalled articles handled?** `isPaywalled` marks member-only articles, and `excludePaywalled` removes them. Only public metadata is collected; the actor never tries to read member-only body content.

**What counts as an article URL?** Standard Medium links, `medium.com/p/...` short links, and articles hosted on a custom Medium domain such as a publication's own website.

**Can I combine filters?** Yes. `fromDate`, `containsKeyword` and `excludePaywalled` work together in every mode.

**Is this affiliated with Medium?** No. This is an independent third-party actor that reads publicly available article metadata. It is not operated by or affiliated with Medium.
