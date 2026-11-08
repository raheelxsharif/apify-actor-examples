# Wayback Machine Snapshot History & Archived Text Extractor

Turn the Internet Archive's Wayback Machine into a queryable dataset. Give the actor a URL, host, or domain and it enumerates every capture the Wayback Machine holds — timestamps, playback links, HTTP status, MIME type, and a content fingerprint — then optionally pulls the readable text of the archived pages themselves.

## What this actor does

- **Four match modes:** `exact` URL, `prefix` (URL and everything beneath it), `host` (one hostname), or `domain` (host plus all subdomains)
- **Date, status, and MIME filters:** narrow by capture date range, HTTP status code (e.g. only archived 404s), or MIME type (e.g. only PDFs)
- **Duplicate control:** collapse adjacent identical captures by content fingerprint, or bucket to one snapshot per month/day/hour
- **Optional full-text retrieval:** download a capped number of the matched snapshots and extract their readable page text
- **Empty fields are omitted** from each record

## Output per snapshot

- `originalUrl` — the URL that was captured
- `timestamp` — the Wayback Machine's raw capture timestamp (`YYYYMMDDHHMMSS`)
- `archiveDate` — the same capture time as an ISO 8601 string
- `archiveUrl` — direct playback link in the Wayback Machine viewer
- `mimeType` — MIME type recorded at capture time
- `statusCode` — HTTP status code recorded at capture time
- `contentDigest` — content fingerprint, used for deduplication
- `contentLength` — captured response body size in bytes
- `content` — extracted readable page text, present only when `includeContent` is enabled and the fetch succeeded
- `contentTruncated` — `true` when the extracted text was cut off at the size cap
- `scrapedAt` — ISO 8601 timestamp of when this record was produced

If no snapshot matches the filters, or the query itself is invalid, the actor emits a single diagnostic record instead (`type: "wayback_search_error"`, with a `reason` of `no_snapshots`, `cdx_fetch_failed`, or `invalid_input`) rather than an empty dataset.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `url` | string | – | **Required.** URL, host, or domain to look up, e.g. `apify.com` or `https://nytimes.com/2024/01/15/story.html` |
| `matchType` | string | `exact` | `exact`, `prefix`, `host`, or `domain` |
| `dateFrom` | string | – | Earliest capture date, inclusive (`YYYY`, `YYYYMM`, or `YYYYMMDD`) |
| `dateTo` | string | – | Latest capture date, inclusive (same formats) |
| `statusFilter` | string | – | Keep only this HTTP status code, e.g. `200` or `404` |
| `mimeFilter` | string | – | Keep only this MIME type, e.g. `text/html` or `application/pdf` |
| `collapseBy` | string | `digest` | `none`, `digest`, `monthly`, `daily`, or `hourly` |
| `maxResults` | integer | `500` | Maximum snapshot records to return (1–10000) |
| `includeContent` | boolean | `false` | Download matched snapshots and extract their readable text |
| `maxContentFetch` | integer | `10` | Cap on how many snapshots to download when `includeContent` is on (0–500) |

### Example: every archived HTML page under a domain, deduplicated

```json
{
    "url": "example.com",
    "matchType": "domain",
    "mimeFilter": "text/html",
    "statusFilter": "200",
    "collapseBy": "digest",
    "maxResults": 2000
}
```

### Example: monthly snapshot of one page through a year

```json
{
    "url": "https://news.ycombinator.com/",
    "matchType": "exact",
    "dateFrom": "2024",
    "dateTo": "2024",
    "collapseBy": "monthly"
}
```

### Example: pull readable text from a page's archived history

```json
{
    "url": "apify.com",
    "matchType": "exact",
    "includeContent": true,
    "maxContentFetch": 20,
    "collapseBy": "digest"
}
```

## Use cases

- **Content audits** — recover every historical version of a page's copy or structure to diff changes over time
- **Broken-link recovery** — enumerate archived 404s on a domain to know exactly what to redirect or restore
- **Competitive tracking** — watch how a competitor's pricing or landing pages evolved month by month
- **Legal and journalism research** — pin down what a page actually said on a specific date
- **Dataset building** — bulk-export archived page text from reference sites for downstream text analysis

## FAQ

**Where does this data come from?**
The Internet Archive's own public Wayback Machine CDX search API — no scraping of the viewer UI involved.

**Do I need a Wayback Machine account or API key?**
No. The CDX search API is free and open; the actor makes no authenticated requests.

**What's the difference between `matchType=host` and `matchType=domain`?**
`host` matches one exact hostname (`www.example.com` won't include `blog.example.com`); `domain` matches that host plus every subdomain.

**Why is `collapseBy=digest` the default?**
Many pages are captured repeatedly with no changes. Collapsing by content fingerprint keeps one record per actual change instead of dozens of identical captures.

**When should I use `monthly`/`daily`/`hourly` instead of `digest`?**
Use a time bucket when you want an evenly spaced sample regardless of content change — one capture per month for a long trend line, one per day for tighter tracking.

**Can I search by keyword instead of URL?**
No. The Wayback Machine's search API indexes by URL/host/domain, not full text. Enable `includeContent` to pull page text, then filter it downstream yourself.

**Why would `content` be missing from a record?**
Either `includeContent` was off, the `maxContentFetch` budget ran out before reaching that snapshot, or the archived page didn't replay successfully.

**Why is `content` sometimes cut short with `contentTruncated: true`?**
Extracted text is capped per page to keep dataset rows manageable; anything beyond the cap is truncated rather than dropped entirely.

**Is archive coverage complete for any URL I search?**
No. The Wayback Machine only has what crawlers happened to capture — popular pages tend to have dense history, obscure ones may have none.

**Is this affiliated with the Internet Archive?**
No. This is an independent, third-party actor that reads the Internet Archive's public Wayback Machine API; it is not operated by the Internet Archive.
