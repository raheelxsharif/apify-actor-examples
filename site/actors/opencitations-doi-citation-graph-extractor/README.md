# OpenCitations DOI Citation Graph Extractor

OpenCitations publishes the largest fully open citation graph of scholarly works, linking papers to everything that cites them and everything they cite. Give this actor a single DOI and it returns the citation graph around that paper — incoming citations, outgoing references, or just a citation count — as structured records with cross-walked identifiers for every connected work.

## What this actor does

- Fetches every work that cites a given DOI (incoming citations)
- Fetches every work a given DOI cites in its own reference list (outgoing references)
- Fetches the total number of works citing a given DOI as a single count record
- Cross-walks each citing/cited work to its DOI, OpenCitations Meta ID (OMID), PubMed ID and OpenAlex ID when OpenCitations has them on file
- Empty fields are omitted from records — a field only appears when OpenCitations has that identifier or flag for the edge

## Output per citation or reference record

- `oci` — Open Citation Identifier, a unique ID for this citation edge
- `citing`, `cited` — raw identifier token strings for the citing and cited works
- `citingDoi`, `citedDoi` — DOI of the citing and cited work
- `citingOmid`, `citedOmid` — OpenCitations Meta ID of each work
- `citingPmid`, `citedPmid` — PubMed ID of each work, when known
- `citingOpenalex`, `citedOpenalex` — OpenAlex ID of each work, when known
- `creation` — date the citing work was published
- `timespan` — ISO-8601 duration between the cited work's and citing work's publication dates
- `journalSc` — `true` when the citation is a journal self-citation (same venue)
- `authorSc` — `true` when the citation is an author self-citation (shared author)
- `journalScTuple` — `{journalSc, authorSc}` combined self-citation flags
- `sourceUrl` — OpenCitations API URL the record came from
- `recordType` — `citation` or `reference`
- `scrapedAt` — ISO timestamp of the scrape

## Output for a citation-count lookup (mode = `getCitationCount`)

- `doi` — the DOI that was looked up
- `citationCount` — number of works citing that DOI
- `sourceUrl` — OpenCitations API URL
- `recordType` — always `citationCount`
- `scrapedAt` — ISO timestamp of the scrape

## Output: no-data / error record (`recordType: "error"`)

Pushed instead of a normal record when the DOI is invalid, or the lookup finds nothing.

- `error` — short code, e.g. `invalid_doi`, `not_found`
- `message` — human-readable explanation
- `input` — the DOI value that caused it
- `sourceUrl` — the OpenCitations API URL that was called
- `scrapedAt`

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `getCitations` | `getCitations` (papers citing the DOI), `getReferences` (papers the DOI cites), or `getCitationCount` (total citing count) |
| `doi` | string | – | The DOI to look up — bare, `doi:`-prefixed, or a full `https://doi.org/…` URL |
| `maxItems` | integer | `50` | Maximum citation/reference records to return (1–1000); ignored for `getCitationCount`, which always returns one record |

### Example: who cites this paper

```json
{
  "mode": "getCitations",
  "doi": "10.1186/1756-8722-6-59",
  "maxItems": 100
}
```

### Example: what this paper cites

```json
{
  "mode": "getReferences",
  "doi": "10.1038/leu.2016.153"
}
```

### Example: just the citation count

```json
{
  "mode": "getCitationCount",
  "doi": "10.1186/1756-8722-6-59"
}
```

## Use cases

- **Literature mapping** — build out the citation network around a seminal paper, one DOI at a time
- **Impact analysis** — track how many works cite a paper and how quickly citations accumulated using `timespan`
- **Reference-list harvesting** — pull a paper's full reference list as structured rows instead of reading a bibliography
- **Self-citation audits** — flag journal and author self-citations at scale using `journalSc`/`authorSc`
- **Identifier reconciliation** — cross-walk a DOI-only dataset to OMID, PMID and OpenAlex identifiers

## FAQ

**Do I need an API key or account?** No. OpenCitations' index API is fully open; no login, key, cookies or proxy are needed.

**What DOI formats does the `doi` field accept?** A bare DOI (`10.1186/1756-8722-6-59`), a `doi:`-prefixed value, or a full `https://doi.org/...` URL — all are normalized automatically.

**What's an OCI?** An Open Citation Identifier — a globally unique, persistent ID OpenCitations assigns to a single citation edge (the link from one work to another).

**Why does a DOI return no results?** Either OpenCitations hasn't indexed that DOI yet, or the work genuinely has no recorded citations or references in the index. In that case the actor still finishes successfully with a single `recordType: "error"` record (`error: "not_found"`) instead of an empty dataset.

**Why are `citingPmid` or `citedOpenalex` sometimes missing?** OpenCitations only stores the identifiers it has successfully cross-walked for a given work. If a work's PubMed ID or OpenAlex ID isn't on file, that field is left out of the record.

**How fresh is the data?** Every run queries the OpenCitations index live, so results reflect whatever OpenCitations has indexed at that moment.

**Is this affiliated with OpenCitations?** No. This is an independent, third-party actor built on the public OpenCitations index API; it is not operated by OpenCitations itself.
