# SEC EDGAR IPO & XBRL Financial Facts Scraper

SEC EDGAR is the US Securities and Exchange Commission's public filing system: every annual report, quarterly report, material-event notice, insider transaction, IPO registration and structured XBRL financial fact from every US-listed company. This actor turns that firehose into clean JSON rows. No cookies, API key, account or proxy are required.

## Four ways to use it

1. **Search filings** (`searchFilings`) — EDGAR full-text search by keyword, form type, date range and optional company (ticker or CIK). Query phrases in quotes. Results are collapsed to one row per filing, so an 8-K with five exhibits is one record, not five.
2. **Browse IPOs** (`browseIPOs`) — the latest S-1 IPO registrations, optionally including S-1/A amendments, filtered by date and keyword.
3. **Company filings** (`getCompanyFilings`) — a company's complete filing history by ticker or CIK. Recent filings come first, then older EDGAR archive pages are fetched automatically when you request more rows. Form-type and date filters are applied while paging.
4. **Company financial facts** (`getCompanyFacts`) — structured XBRL facts (revenue, assets, liabilities, EPS, share counts) with unit, period, fiscal year/period, frame and the accession number of the filing that reported each value. Choose US GAAP, DEI or both. A small `maxItems` returns the latest value of many concepts instead of many historical periods of one concept.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | select | `searchFilings` | searchFilings / getCompanyFilings / getCompanyFacts / browseIPOs |
| `query` | string | – | Full-text query for searchFilings. Quote phrases: `"going concern"`. |
| `formType` | select | `all` | `all`, S-1, S-1/A, S-3, S-4, F-1, F-1/A, 10-K, 10-Q, 8-K, 20-F, 40-F, 6-K, 424B4, 424B3, DEF 14A, proxy statement, 13F-HR, SC 13D, SC 13G, 4, 144, C, D |
| `dateStart` | string | – | Filed on or after this date (`YYYY-MM-DD`). |
| `dateEnd` | string | – | Filed on or before this date (`YYYY-MM-DD`). |
| `cikOrTicker` | string | – | Ticker (AAPL, BRK.B) or 10-digit CIK. Required for company modes; optional company filter for search/browse. |
| `includeAmendments` | boolean | `false` | For browseIPOs, include S-1/A amendments alongside new S-1 registrations. |
| `factTaxonomy` | select | `all` | For getCompanyFacts: `all`, `us-gaap` (financial statements) or `dei` (entity/cover-page data). |
| `maxItems` | integer | `25` | 1–10,000 rows. SEC full-text search itself caps at 10,000 results per query. |

The default prefill is a tiny IPO browse (`maxItems: 3`) so the daily test costs almost nothing and still returns real records.

## Output

Empty values are omitted at every depth — there are no `null`, `""` or `[]` fields.

### Search and IPO rows
`filingId` (accession), `companyName`, `ticker`, `cik`, `formType`, `filingType` (matched document, e.g. `EX-99.1`), `rootFormType`, `filingDate`, `description`, `period`, `fileNumber`, `filmNumber`, `businessLocation`, `incorporationState`, `sicCode`, `items` (8-K item codes), `documentUrl`, `scrapedAt`.

### Company filing rows
`companyName`, `cik`, `ticker`, `sic`, `sicDescription`, `state`, `fiscalYearEnd`, `formType`, `filingDate`, `reportDate`, `acceptanceDateTime`, `accessionNumber`, `fileNumber`, `filmNumber`, `items`, `description`, `documentUrl`, `scrapedAt`.

### XBRL fact rows
`companyName`, `cik`, `taxonomy`, `concept`, `label`, `unit`, `value`, `period`, `start`, `fiscalYear`, `fiscalPeriod`, `frame`, `accessionNumber`, `form`, `filed`, `scrapedAt`.

## Examples

Browse the newest S-1 IPO registrations:
```json
{ "mode": "browseIPOs", "maxItems": 25 }
```

Search 8-K filings that mention "acquisition" in the first half of 2024:
```json
{ "mode": "searchFilings", "query": "acquisition", "formType": "8-K", "dateStart": "2024-01-01", "dateEnd": "2024-06-30", "maxItems": 50 }
```

Search only Apple's 10-K filings for "artificial intelligence":
```json
{ "mode": "searchFilings", "query": "\"artificial intelligence\"", "formType": "10-K", "cikOrTicker": "AAPL", "maxItems": 20 }
```

Pull 1,100 recent Apple filings (crosses from the recent page into EDGAR archive pages):
```json
{ "mode": "getCompanyFilings", "cikOrTicker": "AAPL", "maxItems": 1100 }
```

Get Apple's latest 50 US GAAP facts:
```json
{ "mode": "getCompanyFacts", "cikOrTicker": "AAPL", "factTaxonomy": "us-gaap", "maxItems": 50 }
```

Browse crowdfunding Form C filings:
```json
{ "mode": "searchFilings", "formType": "C", "dateStart": "2024-01-01", "maxItems": 50 }
```

## Use cases

- **IPO tracking** — watch new S-1 registrations and amendments before they reach news feeds.
- **Investment research** — pull annual/quarterly reports and XBRL fundamentals for a watchlist.
- **Crowdfunding deal flow** — browse Reg CF Form C and private-placement Form D filings.
- **M&A and event intelligence** — search 8-K items for merger, acquisition and material-event language.
- **Compliance and due diligence** — verify filing history, SIC code, state of incorporation and fiscal year end.
- **Insider and institutional activity** — filter Forms 3/4/5, 144, 13F-HR, SC 13D and SC 13G.
- **Academic research** — reproducible, source-linked XBRL panels without paid terminals.

## Limits and data notes

- **10,000-result search cap.** SEC's full-text index returns at most 10,000 matches per query. Narrow by form, date or company to reach older results. The status message says when the cap applies.
- **Full-text matching is document-level.** `filingType` can be an exhibit (`EX-99.1`) when the keyword matched inside that exhibit; `formType`/`rootFormType` still identify the filing itself.
- **XBRL starts around 2009.** Companies that never filed XBRL have no `getCompanyFacts` rows.
- **No primary-document text.** The actor returns filing metadata, links and XBRL facts; it does not download or parse full filing HTML.
- **Rate limiting.** Requests are paced under SEC's 10 requests/second guidance and retried on 429/5xx.

## Legacy inputs

Saved tasks from the earlier version still run: `companies`, `filingTypes`, `dateRangeFrom`, `dateRangeTo`, `maxItemsPerCompany`, `containsKeyword` and `userAgentEmail` are read in code and mapped onto the new fields. `fetchPrimaryDocText` is no longer supported and is logged as ignored. The input form shows only the new fields.

## FAQ

**Does it need a login, API key or cookies?** No. EDGAR is fully public.

**Is a proxy required?** No. The actor calls SEC's public JSON endpoints directly.

**How fresh is the data?** SEC publishes filings within minutes; the actor reads the same live index.

**What is a CIK?** SEC's numeric company identifier. Tickers and CIKs are both accepted, including class-share tickers such as `BRK.B`.

**Can I get private-company financials?** Only what the company filed publicly (usually Form D or Reg CF), and only companies that file with the SEC.

**Is this affiliated with the SEC?** No. This is an independent tool, not endorsed by or affiliated with the U.S. Securities and Exchange Commission.
