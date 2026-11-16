# OpenCitations DOI Citation Graph Extractor

OpenCitations publishes a fully open citation graph of scholarly works. Give this actor one or more DOIs — or PMIDs, OMIDs, ISSNs, ISBNs — and it returns the graph around them: the papers that cite each work, the papers each work cites, both directions in one run, citation and reference totals, single citation edges, and full bibliographic metadata. Everything comes from OpenCitations' public Index and Meta APIs, so no key, login, cookie or proxy is needed.

## Modes

| Mode | What it returns |
|---|---|
| `getCitations` | One row per work that cites each identifier (incoming citations) |
| `getReferences` | One row per work each identifier cites (outgoing references) |
| `getGraph` | Citations and references together, split evenly by `maxItems` |
| `getCitationCount` | Incoming citation total for each identifier |
| `getReferenceCount` | Outgoing reference total for each identifier |
| `getCounts` | Citation and reference totals in one record (plus venue count for an ISSN) |
| `getVenueCitationCount` | All incoming citations to a journal identified by ISSN |
| `getMetadata` | Title, authors, venue, publisher, volume, issue, pages, type and dates |
| `getCitation` | A single citation edge looked up by its OCI |

## Inputs

| Field | Type | Default | Notes |
|---|---|---|---|
| `mode` | select | `getCitations` | One of the modes above |
| `identifiers` | string list | `[]` | DOIs, DOI URLs, `doi:` values, PMIDs, OMIDs, ISSNs or ISBNs; one per line |
| `oci` | string | empty | Only for `getCitation`, e.g. `06101801781-06180334099` |
| `maxItems` | integer | `100` (prefill `3`) | Row cap for edge modes; identifier cap for count and metadata modes |
| `startPage` | integer | `1` | Resume or slice a large edge result set; each page holds up to 1,000 edges |
| `includeWorkMetadata` | boolean | `false` | Add `citingTitle`, `citingAuthors`, `citingPubDate`, `citingVenue` and the cited equivalents to edge rows |

Saved tasks that still send the old `doi` key keep working: the actor maps it onto `identifiers` in code. The old `mode`, `maxItems` and output fields are unchanged.

### Examples

Who cites a paper:

```json
{
  "mode": "getCitations",
  "identifiers": ["10.1186/1756-8722-6-59"],
  "maxItems": 100
}
```

Both directions for two papers:

```json
{
  "mode": "getGraph",
  "identifiers": [
    "https://doi.org/10.1038/leu.2016.153",
    "10.7717/peerj-cs.421"
  ],
  "maxItems": 200
}
```

Counts and metadata:

```json
{
  "mode": "getCounts",
  "identifiers": ["10.1186/1756-8722-6-59", "issn:0138-9130"]
}
```

```json
{
  "mode": "getMetadata",
  "identifiers": ["pmid:23958373", "10.7717/peerj-cs.421"]
}
```

A single edge by OCI:

```json
{
  "mode": "getCitation",
  "oci": "06101801781-06180334099"
}
```

## Output

Records are flat and omit-empty: a field appears only when OpenCitations has a real value for it.

### Citation and reference edge

- `oci` — Open Citation Identifier for the edge
- `queryIdentifier` — the identifier you asked for
- `resolvedIdentifier` — the DOI/PMID/OMID used after resolving an OpenAlex or PMCID input
- `citing`, `cited` — raw identifier token strings
- `citingDoi`, `citedDoi` — DOI of each side
- `citingOmid`, `citedOmid` — OpenCitations Meta ID of each side
- `citingPmid`, `citedPmid` — PubMed ID of each side, when known
- `citingOpenalex`, `citedOpenalex` — OpenAlex ID of each side, when known
- `citingUrl`, `citedUrl` — DOI resolver links
- `citingPubmedUrl`, `citedPubmedUrl` — PubMed links, when a PMID is known
- `creation` — publication date of the citing work
- `timespan` — ISO-8601 duration between the cited and citing publication dates
- `journalSc`, `authorSc` — journal / author self-citation flags
- `journalScTuple` — the two flags together
- `citingTitle`, `citingAuthors`, `citingPubDate`, `citingVenue` — only with `includeWorkMetadata`
- `citedTitle`, `citedAuthors`, `citedPubDate`, `citedVenue` — only with `includeWorkMetadata`
- `sourceUrl` — exact OpenCitations page URL
- `recordType` — `citation` or `reference`
- `scrapedAt` — ISO timestamp

### Counts

- `identifier`, plus `doi` / `issn` / `pmid` / `omid` as applicable
- `citationCount` — incoming citations
- `referenceCount` — outgoing references (`getCounts`, `getReferenceCount`)
- `venueCitationCount` — citations to an ISSN (`getCounts` when the identifier is an ISSN, `getVenueCitationCount`)
- `sourceUrl` — API URL used
- `recordType` — `citationCount`, `referenceCount`, `venueCitationCount` or `counts`
- `scrapedAt`

### Metadata

- `identifier`, `doi`, `omid`, `pmid`, `pmcid`, `openalexId`, `issn`, `issns`, `isbn`
- `title`, `author`, `authors` (name + ORCID/OMID when known)
- `pubDate`, `publicationYear`
- `volume`, `issue`, `page`, `publicationType`
- `venue`, `venueName`, `venueIssn`, `venueOmid`, `venueOpenalex`
- `publisher`, `publisherName`, `publisherCrossref`, `publisherOmid`
- `editor`, `editors`
- `doiUrl`, `sourceUrl`
- `recordType` — `metadata`
- `scrapedAt`

### Error row

Pushed instead of a normal record for an invalid identifier, an unresolved identifier, an empty lookup, or an API failure.

- `error` — short code such as `invalid_identifier`, `invalid_oci`, `missing_identifier`, `unresolved_identifier`, `not_found` or `fetch_failed`
- `message` — plain-language explanation
- `input` — the value that caused it
- `mode` — the mode being run
- `sourceUrl`
- `recordType` — `error`
- `scrapedAt`

## Pagination and large graphs

Citation and reference modes use OpenCitations' server-side `page` / `page_size` parameters. The actor keeps requesting pages until `maxItems` is reached or the source runs out, deduplicates edges by OCI, and never treats the natural end of results as a failure. `startPage` lets you resume a very large graph in slices; `maxItems` is the total row budget across all identifiers in the run.

## Data source and scope

- Citations, references, counts and OCIs: OpenCitations Index API v2 (`api.opencitations.net/index/v2`).
- Titles, authors, venues, publishers and dates: OpenCitations Meta API v1 (`api.opencitations.net/meta/v1`).
- OpenCitations rate-limits anonymous API use to 180 requests per minute per IP. The actor paces its page requests; extremely large graphs can take a while.
- The Index contains citation links, not full texts or abstracts.
- OpenAccess/PMCID and OpenAlex identifiers are accepted directly by `getMetadata`; in edge and count modes the actor resolves them to a DOI/OMID/PMID through Meta first.
- Metadata is only as complete as the OpenCitations record: missing authors, venues or pages are omitted rather than guessed.
- This is an independent third-party actor. It is not operated by or affiliated with OpenCitations.

## FAQ

**Do I need an API key?** No. The OpenCitations APIs used here are open and cookieless.

**Can I look up several papers in one run?** Yes — put one identifier per line in `identifiers`. `maxItems` is shared across all of them, and each row carries `queryIdentifier` so you can group the output.

**What identifier formats are accepted?** Bare DOIs, `doi:` values, `https://doi.org/…` URLs, PMIDs, `omid:br/…`, `issn:…`, `isbn:…` and, when OpenCitations has them, `openalex:W…` and `pmcid:PMC…` for metadata. Bare 8-digit values are treated as PMIDs; use `issn:`/`isbn:` prefixes when that is what you have.

**Why is a citation row missing a PMID or OpenAlex ID?** OpenCitations only stores the identifiers it has cross-walked for that work. Missing identifiers are omitted.

**Why does `getGraph` return fewer citations than `getCitations` with the same `maxItems`?** `getGraph` splits the row budget between directions, so both halves appear in one dataset. Run `getCitations` or `getReferences` for a full single-direction budget.

**Can I get more than 100,000 rows?** `maxItems` is capped at 100,000 per run. Use `startPage` to continue a larger result set in another run.

**How fresh is the data?** Every run queries OpenCitations live, so results reflect the current index.
