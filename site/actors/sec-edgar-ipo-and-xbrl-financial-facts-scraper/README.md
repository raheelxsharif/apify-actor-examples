# SEC EDGAR IPO & XBRL Financial Facts Scraper

SEC EDGAR is the US Securities and Exchange Commission's public database of every regulatory filing from every US-listed company. This actor searches that database by keyword and form type, browses new IPO registrations (S-1), pulls a specific company's full filing history, or extracts its structured XBRL financial facts such as revenue and assets. No account is required.

## What this actor does

**4 modes in one actor:**

1. **Search filings** (`searchFilings`) — full-text keyword search across all EDGAR filings, filtered by form type and date range
2. **Browse IPOs** (`browseIPOs`) — browse recent S-1 registration statements (IPO filings) by date and keyword
3. **Company filings** (`getCompanyFilings`) — fetch all recent filings for a specific company by ticker or CIK
4. **Company financial facts** (`getCompanyFacts`) — extract structured XBRL financial data (revenue, assets, liabilities, earnings per share, etc.)

## Output fields

### searchFilings / browseIPOs
| Field | Description |
|---|---|
| `filingId` | SEC accession number (e.g. `0001234567-24-000001`) |
| `companyName` | Entity name |
| `ticker` | Stock ticker if available |
| `cik` | 10-digit padded CIK |
| `formType` | Filing form (S-1, 10-K, 8-K, etc.) |
| `filingDate` | Date filed (YYYY-MM-DD) |
| `description` | Filing description |
| `documentUrl` | Direct link to EDGAR filing index |
| `period` | Reporting period end date |
| `scrapedAt` | Scrape timestamp |

### getCompanyFilings
One record per filing: `companyName`, `cik`, `ticker`, `sic`, `sicDescription`, `state`, `fiscalYearEnd`, `formType`, `filingDate`, `accessionNumber`, `documentUrl`, `description`, `scrapedAt`

### getCompanyFacts
One record per financial data point: `companyName`, `cik`, `concept`, `label`, `unit`, `value`, `period`, `form`, `filed`, `scrapedAt`

Empty fields are omitted (no nulls anywhere).

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | select | `searchFilings` | One of: searchFilings / getCompanyFilings / getCompanyFacts / browseIPOs |
| `query` | string | – | Full-text search query (searchFilings / browseIPOs) |
| `formType` | select | `all` | S-1 / 10-K / 10-Q / 8-K / C / D / proxy statement / all |
| `dateStart` | string | – | Filter filings on or after this date (YYYY-MM-DD) |
| `dateEnd` | string | – | Filter filings on or before this date (YYYY-MM-DD) |
| `cikOrTicker` | string | – | Ticker (AAPL) or CIK (0000320193) — required for company modes |
| `maxItems` | integer | `25` | Max records to return (1–200) |

### Example: Browse recent IPO filings
```json
{
  "mode": "browseIPOs",
  "maxItems": 25
}
```

### Example: Search 8-K filings mentioning "acquisition"
```json
{
  "mode": "searchFilings",
  "query": "acquisition",
  "formType": "8-K",
  "dateStart": "2024-01-01",
  "maxItems": 50
}
```

### Example: Get Apple's recent SEC filings
```json
{
  "mode": "getCompanyFilings",
  "cikOrTicker": "AAPL",
  "maxItems": 25
}
```

### Example: Get Apple's financial facts (XBRL)
```json
{
  "mode": "getCompanyFacts",
  "cikOrTicker": "AAPL",
  "maxItems": 100
}
```

### Example: Browse crowdfunding filings (Reg CF)
```json
{
  "mode": "searchFilings",
  "formType": "C",
  "dateStart": "2024-01-01",
  "maxItems": 50
}
```

## Use cases

- **IPO tracking** — monitor new S-1 registrations in real time; identify upcoming listings before they hit news feeds
- **Investment research** — pull annual/quarterly reports for a watchlist; extract XBRL financials for quantitative analysis
- **Crowdfunding deal flow** — browse Form C filings from companies raising capital under Regulation Crowdfunding
- **Private placement monitoring** — scan Form D filings for new exempt offerings and venture rounds
- **Compliance & due diligence** — verify filing history, SIC codes, state of incorporation, fiscal year end
- **M&A intelligence** — search 8-K filings mentioning "merger", "acquisition", or specific company names
- **Financial journalism** — access public disclosures programmatically without paying for Bloomberg or FactSet
- **Academic research** — bulk-download structured XBRL financial data for empirical finance studies

## FAQ

**Does it require a login, API key, or cookies?**
No. SEC EDGAR is fully public. This actor requires zero credentials.

**Is a proxy needed?**
No. SEC EDGAR's public filing data is accessible from any location without a proxy.

**What is the rate limit?**
SEC asks that automated tools stay under roughly 10 requests per second; the actor's own request pace is designed to stay within that limit.

**How fresh is the data?**
Close to real-time. SEC typically makes a filing available within minutes of a company submitting it.

**What is an accession number?**
SEC's unique filing identifier, formatted as `XXXXXXXXXX-YY-ZZZZZZ` (filer-year-sequence). Each document submitted to EDGAR has a unique accession number.

**What is a CIK?**
Central Index Key — SEC's unique numeric identifier for each filer. Every public company, fund, and individual who files with the SEC has a CIK. The actor accepts both tickers and CIKs.

**What are XBRL financial facts?**
eXtensible Business Reporting Language facts are structured, machine-readable financial data extracted from filings. The `getCompanyFacts` mode returns individual data points like `RevenueFromContractWithCustomerExcludingAssessedTax` (revenue) with value, period, and filing provenance.

**Can I get financial data for private companies?**
No — only companies that file with the SEC (publicly listed companies and certain private filers). Truly private companies do not file with the SEC except for Form D (exempt offering notices) and Reg CF (crowdfunding).

**What is the difference between S-1 and Form C?**
S-1 is an IPO registration statement for companies going public on a major exchange (NYSE, Nasdaq). Form C is a crowdfunding offering circular for companies raising up to $5M from the general public under Regulation Crowdfunding — typically early-stage startups.

**Is this affiliated with the SEC?**
No. This is an independent tool and is not affiliated with, endorsed by, or sponsored by the US Securities and Exchange Commission.
