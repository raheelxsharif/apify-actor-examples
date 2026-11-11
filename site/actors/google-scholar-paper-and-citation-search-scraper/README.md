# Google Scholar Paper & Citation Search Scraper

Search **Google Scholar** for papers, follow their citations, and export public researcher profiles as clean structured data. One actor covers keyword search with year and review filters, citing-paper lists, author metrics such as h-index and i10-index, PDF links, citation styles, and cluster identifiers for follow-up queries. No Google account, API key, or proxy setup is needed.

## What this actor does

- **Three modes:** `search` finds papers by keyword, `citations` lists papers that cite a known article, and `authorProfile` exports a public researcher profile with its publication list.
- **Paper search filters:** year range, newest-first sorting, review articles only, and client-side PDF/HTML filtering. Optional advanced filters narrow results by author, venue, or title-only matching.
- **Citing papers:** pass a Scholar cluster ID or a `cites=` URL and receive the citing works with the same paper fields.
- **Author profiles:** name, affiliation, verified email domain, homepage, research interests, citations, h-index, i10-index (all-time and recent), plus the paginated publication table.
- **Citation styles:** optionally add MLA, APA, Chicago, Harvard, and Vancouver text for returned results, with a bounded time budget so search rows are still saved if formats are slow.
- **Honest data:** PDF links point at the file Scholar lists, book and citation-only rows are typed correctly, duplicate rows within each query or citation list are removed, and empty fields are omitted.

## Output per record

Every record has a `recordType`: `paper`, `citation`, or `author`.

### Paper and citation records

- `title` — paper or article title
- `link` — publisher or landing-page URL
- `documentLink` — direct PDF or HTML file URL, when Scholar lists one
- `documentType` — `PDF`, `HTML`, `PS`, or omitted
- `authors` — author names as shown by Scholar
- `publication` — journal, conference, or venue text from Scholar (some rows include the publisher or source domain)
- `year` — publication year
- `source` — hosting domain, such as `arxiv.org`, when Scholar separates it from the venue
- `fullAttribution` — the complete attribution line from Scholar
- `searchMatch` — snippet showing where the query matched
- `citations` — cited-by count (`0` when Scholar shows none)
- `citationsLink` — link to the citing-paper list
- `relatedArticlesLink` — link to related articles
- `versions` / `versionsLink` — version count and link, when Scholar shows them
- `type` — `ARTICLE`, `BOOK`, or `CITATION`
- `resultIndex` — position in the result list (0-based)
- `clusterId` — Scholar cluster ID, useful for citation lookups
- `resultId` — Scholar's internal result ID
- `scholarUrl` — Scholar page for the article cluster
- `authorIds` / `authorProfileUrls` — linked author profiles
- `doi` — clean DOI parsed from the result URL when present; Scholar does not expose a DOI field directly
- `citationFormats` — `mla`, `apa`, `chicago`, `harvard`, `vancouver` when requested
- `searchQuery` — the query that found the paper (search mode)
- `citedByOf` — the cluster ID being cited (citations mode)
- `scrapeTimestamp` — ISO 8601 scrape time

### Author records

- `authorId`, `profileUrl`, `name`, `affiliation`
- `verifiedEmailDomain`, `homepageUrl`, `interests`, `thumbnailUrl`
- `citations`, `citationsSince`, `hIndex`, `hIndexSince`, `i10Index`, `i10IndexSince`, `metricsSinceYear`
- `publications` — title, authors, venue, year, citations, Scholar URL, citation ID, and optional journal/volume/issue/pages/publisher/abstract/publication date/total citations
- `publicationCount`
- `scrapeTimestamp`

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `search` | `search`, `citations`, or `authorProfile` |
| `queries` | array | `["attention is all you need"]` | Search terms for `mode=search` |
| `clusterIds` | array | – | Cluster IDs or `cites=` URLs for `mode=citations` |
| `authorIds` | array | – | Author IDs or profile URLs for `mode=authorProfile` |
| `maxItems` | integer | `100` | Maximum papers per query/cluster, or papers per author (1–1000) |
| `sortBy` | string | `relevance` | `relevance` or `date` |
| `filter` | string | `all` | `all`, `pdfOnly`, or `htmlOnly` |
| `articleType` | string | `any` | `any` or `review` |
| `newerThan` | integer | – | Earliest publication year |
| `olderThan` | integer | – | Latest publication year |
| `author` | string | – | Return papers by this author |
| `publication` | string | – | Return papers from this venue |
| `titleOnly` | boolean | `false` | Match the query against titles |
| `includeCitationFormats` | boolean | `false` | Add five citation styles for returned rows within a bounded budget; a slow format lookup never drops the search rows |
| `includePaperDetails` | boolean | `false` | Add journal, volume, issue, pages, publisher, publication date, abstract, and total citations to author publications |

## Examples

### Search recent review papers on a medical topic

```json
{
    "mode": "search",
    "queries": ["cancer immunotherapy"],
    "maxItems": 30,
    "articleType": "review",
    "newerThan": 2022
}
```

### Find papers by a specific author in a specific venue

```json
{
    "mode": "search",
    "queries": ["quantum computing"],
    "author": "Andrew Steane",
    "publication": "Physical Review",
    "maxItems": 20
}
```

### List papers that cite a known article

```json
{
    "mode": "citations",
    "clusterIds": ["184251978817409129"],
    "maxItems": 50,
    "includeCitationFormats": true
}
```

### Export an author profile with paper details

```json
{
    "mode": "authorProfile",
    "authorIds": ["https://scholar.google.com/citations?user=-AXbSiwAAAAJ&hl=en"],
    "maxItems": 100,
    "includePaperDetails": true
}
```

## Use cases

- **Literature reviews** — collect a structured reading list and follow citation chains without copying rows from Scholar.
- **Bibliometrics** — track citation counts, h-index, and i10-index for researchers or fields.
- **Research monitoring** — rerun saved queries to catch new papers in a topic.
- **Citation analysis** — export citing papers for one or more key articles.
- **Academic recruiting and grants** — compare public publication records and recent citation metrics.
- **Data pipelines** — feed clean JSON into a database, spreadsheet, or research assistant.

## FAQ

### Do I need a Google Scholar account, API key, or proxy?

No. The actor reads public Scholar pages and selects a working connection automatically. Just provide the input and run it.

### Why do I get fewer results than `maxItems`?

Google Scholar serves about 10 search results per page and may return an empty or partial page for deep pagination. The actor retries on a fresh connection and stops with an honest status instead of inventing rows. For broad topics, split the query by year, venue, or author to reach more results.

### What is a cluster ID and where do I find it?

A cluster ID identifies one article and all of its known versions on Scholar. It is the number in a `cites=` or `cluster=` URL, and it is also returned as `clusterId` on every search result. Use it as input to `mode=citations` to list citing papers.

### Why is a DOI missing from some records?

Google Scholar does not show a DOI field on search results. The actor adds a DOI only when the result's URL contains one, so many records correctly omit it. It never guesses a DOI. Query strings, fragments, and common suffix paths such as `/meta` or `.html` are removed.

### Which citation styles are supported?

MLA, APA, Chicago, Harvard, and Vancouver are returned as text when `includeCitationFormats` is enabled. The actor looks up each returned result ID, including deeper pages, within a bounded time budget. If Google Scholar is slow or blocks some format lookups, the already-collected paper rows are still saved and the run status reports how many rows received formats. The option adds one request per row, so it is slower than a plain search.

### Why are some PDF links missing?

Scholar only shows a file link when it has indexed a PDF or HTML copy. The actor returns the real file URL and omits `documentLink` when no file is available. `link` still points to the publisher or landing page.

### Why do author profiles not include coauthors?

Coauthor cards are not part of the public profile view; Scholar only exposes them through a signed-in editing dialog. The actor returns the public fields it can verify: affiliation, interests, metrics, and the publication list.

### Can I search for an author by name and get their profile automatically?

Author-name profile search requires a Google sign-in, so the actor does not offer it. If you know the profile URL or user ID, paste it into `authorIds`; author names linked from search results are also returned in `authorIds` and `authorProfileUrls`.

### How are year filters handled with newest-first sorting?

When a year range is set, the actor sends Scholar's server-side `as_ylo`/`as_yhi` bounds (even with `sortBy: "date"`) and verifies the years on the returned rows. Newest-first order is then applied within each page, because Scholar ignores a global date sort when a year range is active. A 2020–2022 range therefore returns matching papers instead of a false empty result.

### Is this affiliated with Google or Google Scholar?

No. This is an independent third-party actor that reads publicly available Google Scholar pages. It is not operated by or endorsed by Google.
