# Wayback Machine Snapshot History & Archived Text Extractor

Turn the Internet Archive's public Wayback Machine into a structured dataset. Give this actor one URL, a host, an entire domain, or a bulk list, then choose a capture history, a version timeline, an archive-coverage summary, the closest snapshot to a date, or the readable text of the archived page itself.

No account, API key, cookie, or proxy setup is needed. The actor talks to the same public CDX and availability endpoints that power the Wayback Machine website.

## Modes

| Mode | What you get | Typical use |
|---|---|---|
| `search` | One row per archived capture with timestamp, playback URL, HTTP status, MIME type, digest, and size | Full capture history, broken-link recovery, redirect mapping |
| `timeline` | One row per distinct content version, with first/last seen dates, capture count, and duration | See how a page changed without reading every duplicate capture |
| `summary` | One archive-coverage row per target: scanned capture count, version count, first/last capture, days covered, status and year breakdowns | Vet expired domains, measure archive density, compare sites |
| `closest` | The capture nearest to a target date, with its playback URL and distance in days | Legal, compliance, and journalism checks: "what did the page say on this date?" |
| `content` | Readable text plus title, description, language, canonical URL, headings, word count, character count, and text hash from archived HTML | Recover deleted copy, search archived text, build text corpora |

## Bulk targets and pagination

- Put one target in `url` or many in `urls`; both are merged and deduplicated.
- The `maxItems` budget is split evenly across targets so every target gets a fair share.
- Capture history uses automatic resume-key pagination, so a single run can retrieve up to 100,000 captures instead of stopping at the first CDX page.
- `sortDirection` chooses whether the limited result set comes from the oldest or newest end of the archive. Search rows are chronological; `latest` reverses the final rows so the newest capture appears first.
- `collapseBy` applies to `search` mode: keep every raw capture, one per distinct content digest, one per URL, or one per month/day/hour. `timeline` and `summary` always read raw captures so version durations and coverage counts are not flattened.
- `summary` scans up to `maxItems` captures per target, split between the oldest and newest ends, so the first and last capture stay accurate even on dense domains. `captureCount`, `versionCount`, and the breakdowns describe that scanned window; raise `maxItems` for a fuller census.

## Output fields

### Capture rows (`search`)

`originalUrl`, `urlKey`, `timestamp`, `archiveDate`, `archiveUrl`, `rawArchiveUrl`, `sourceUrl`, `statusCode`, `statusText`, `statusClass`, `mimeType`, `contentDigest`, `contentLength`, `host`, `urlPath`, `fileExtension`, `captureYear`, `matchType`, `isRevisit`, `recordType`, `resultIndex`, `scrapedAt`.

With `detectChanges` on, each row also gets `captureIndex`, `changed`, `changeType` (`initial`, `content`, `status`, `contentAndStatus`, `unchanged`), `isInitialCapture`, `previousTimestamp`, `previousStatusCode`, `previousContentDigest`, and `daysSincePrevious`.

With `enrichWithContent` on, rows also get the archived-text fields below, up to `maxContentFetch` rows.

### Version rows (`timeline`)

`originalUrl`, `firstTimestamp`, `lastTimestamp`, `firstArchiveDate`, `lastArchiveDate`, `captureCount`, `durationDays`, `archiveUrl`, `rawArchiveUrl`, `contentDigest`, `mimeType`, `statusCode`, `statusText`, `contentLength`, `sourceUrl`, `recordType`, `resultIndex`, `scrapedAt`.

### Summary rows (`summary`)

`originalUrl`, `matchType`, `archived`, `captureCount`, `versionCount`, `firstTimestamp`, `lastTimestamp`, `firstArchiveDate`, `lastArchiveDate`, `firstArchiveUrl`, `lastArchiveUrl`, `spanDays`, `statusBreakdown`, `mimeTypeBreakdown`, `activeYears`, `yearBreakdown`, `sourceUrl`, `recordType`, `resultIndex`, `scrapedAt`.

### Closest-snapshot rows (`closest`)

`originalUrl`, `available`, `requestedTimestamp`, `requestedDate`, `timestamp`, `archiveDate`, `archiveUrl`, `rawArchiveUrl`, `statusCode`, `statusClass`, `statusText`, `captureDistanceDays`, `sourceUrl`, `recordType`, `resultIndex`, `scrapedAt`.

### Archived-page rows (`content`)

`originalUrl`, `timestamp`, `archiveDate`, `archiveUrl`, `rawArchiveUrl`, `statusCode`, `mimeType`, `contentDigest`, `contentLength`, `title`, `description`, `language`, `canonicalUrl`, `metaKeywords`, `headings`, `text`, `wordCount`, `characterCount`, `textHash`, `contentTruncated`, `fetchContentLength`, `fetchContentTruncated`, `contentMimeType`, `sourceUrl`, `recordType`, `resultIndex`, `scrapedAt`.

Empty values are omitted at every depth, so a field appears only when the archive actually supplied it.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `search` | `search`, `timeline`, `summary`, `closest`, or `content` |
| `url` | string | – | Single URL, host, domain, or playback URL |
| `urls` | array | `[]` | Bulk targets; merged with `url` |
| `matchType` | string | `exact` | `exact`, `prefix`, `host`, or `domain`; wildcards are auto-detected |
| `dateFrom` | string | – | Earliest capture date, inclusive |
| `dateTo` | string | – | Latest capture date, inclusive |
| `statusCodes` | array | `[]` | Keep only these HTTP status codes |
| `excludeStatusCodes` | array | `[]` | Drop these HTTP status codes |
| `mimeTypes` | array | `[]` | Keep only these MIME types |
| `excludeMimeTypes` | array | `[]` | Drop these MIME types |
| `collapseBy` | string | `digest` | `none`, `digest`, `urlkey`, `monthly`, `daily`, or `hourly`; applies to `search` only |
| `sortDirection` | string | `earliest` | `earliest` or `latest` |
| `targetDate` | string | – | Reference date for `closest` and `content`; defaults to now |
| `enrichWithContent` | boolean | `false` | Add archived text to search rows |
| `maxContentFetch` | integer | `10` | Text-fetch cap for search enrichment (0–500) |
| `maxContentChars` | integer | `500000` | Text cap per archived page (1,000–2,000,000) |
| `includeHeadings` | boolean | `true` | Include H1–H6 headings in archived text |
| `maxItems` | integer | `500` | Record cap (1–100,000) |
| `detectChanges` | boolean | `true` | Annotate content/status changes in search rows |

The daily-test prefill is the cheapest possible run: `mode=closest`, `url=apify.com`, `targetDate=2024-01-01`, `maxItems=1`. It performs one availability lookup and returns one snapshot record.

### Example: full capture history with changes

```json
{
    "mode": "search",
    "url": "https://example.com/",
    "matchType": "exact",
    "dateFrom": "2020-01-01",
    "dateTo": "2024-12-31",
    "statusCodes": ["200"],
    "collapseBy": "none",
    "detectChanges": true,
    "maxItems": 500
}
```

### Example: bulk closest snapshots

```json
{
    "mode": "closest",
    "urls": ["apify.com", "example.com", "wikipedia.org"],
    "targetDate": "2020-06-15",
    "maxItems": 3
}
```

### Example: archived text from a playback URL

```json
{
    "mode": "content",
    "url": "https://web.archive.org/web/20240101000000/https://apify.com/",
    "maxItems": 1,
    "includeHeadings": true
}
```

## Legacy inputs

Saved tasks and integrations from earlier versions keep working. The actor still accepts `url` as a single target. Old field names `maxResults`, `includeContent`, `statusFilter`, and `mimeFilter` are mapped onto `maxItems`, `enrichWithContent`, `statusCodes`, and `mimeTypes` in code. Collapse aliases `timestamp:6`, `timestamp:8`, and `timestamp:10` are accepted alongside `monthly`, `daily`, and `hourly`.

## Use cases

- **Content recovery** — pull the text of pages that no longer exist
- **SEO link-rot audits** — list archived 404s, redirects, and deleted pages on a domain
- **Competitive history** — compare pricing, messaging, or product pages month by month
- **Legal and journalism evidence** — pin the closest capture to a specific date
- **Domain due diligence** — measure archive age, density, gaps, and version count before buying an expired domain
- **Dataset building** — export archived text and metadata for search, NLP, or research

## FAQ

**Where does the data come from?**
The Internet Archive's public Wayback CDX search API, its availability API, and archived playback pages. No browser UI is scraped.

**Do I need an Internet Archive account or API key?**
No. Every request is anonymous and cookieless.

**Can I search the archive by keyword?**
The CDX API indexes URLs and capture metadata, not full text. Use `mode=content` to download page text and filter it in your own pipeline.

**Why are some captures marked `warc/revisit`?**
The archive sometimes records a revisit instead of a full copy. Those rows keep the digest and size but may not have an HTTP status code.

**Why is `content` missing from a row?**
The row was beyond the text-fetch budget, the archived resource was not HTML/text, or replay returned an error. `contentTruncated` is set when text was clipped at `maxContentChars`.

**Why does `closest` mode omit MIME type and digest?**
The availability endpoint returns the nearest capture URL, timestamp, and status only. Use `search` mode with a narrow date range when you need full CDX metadata.

**Can I retrieve more than 10,000 captures?**
Yes. The actor pages through the CDX index with resume keys, so `maxItems` can go up to 100,000. Very broad host/domain queries and `collapseBy=urlkey` can be slower on the archive side.

**Is archive coverage complete?**
No. The Wayback Machine only stores what its crawlers captured. Popular pages usually have dense history; obscure pages may have none, and a summary row with `captureCount=0` is still a valid result.

**Is this affiliated with the Internet Archive?**
No. It is an independent tool that reads the Internet Archive's public Wayback Machine endpoints.
