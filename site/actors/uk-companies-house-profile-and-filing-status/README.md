# UK Companies House Profile & Filing Status Scraper

Companies House is the UK government's official registrar of companies. This actor searches it by company name or looks up specific registration numbers directly, returning each company's registration status, registered office address, SIC industry codes, incorporation date, and its accounts and confirmation-statement filing schedule.

## What this actor does

- Searches companies by name or free-text keywords, with optional type and status filters
- Looks up full profiles for specific company registration numbers in one run
- Returns registered office address, SIC industry codes, and jurisdiction
- Returns the accounts and confirmation-statement due/last-filed dates, plus charge and insolvency-history flags
- Only fields Companies House actually has on file for a company are included — nothing is sent as a placeholder

## Authentication

This actor requires a free Companies House API key, passed as the `apiKey` input. Register at [developer.company-information.service.gov.uk](https://developer.company-information.service.gov.uk) — registration takes a few minutes. Without a key, the run completes with an empty dataset and a status message explaining how to get one, instead of failing.

## Output per company

- `companyNumber` — unique UK company registration number
- `companyName` — registered company name
- `companyType` — legal type, e.g. `ltd`, `plc`, `llp`
- `companyStatus` — current status, e.g. `active`, `dissolved`, `liquidation`
- `dateOfCreation` — incorporation date
- `dateOfCessation` — date the company ceased, when dissolved
- `registeredOfficeAddress` — address lines, locality, region, postal code, and country
- `sicCodes[]` — Standard Industrial Classification codes for the company's business activity
- `accounts` — `{nextDue, lastMadeUpTo, ...}` accounts filing dates
- `confirmationStatement` — `{nextDue, lastMadeUpTo}` confirmation statement dates
- `jurisdiction` — legal jurisdiction, e.g. `england-wales`, `scotland`
- `hasCharges` — whether the company has registered charges
- `hasInsolvencyHistory` — whether the company has insolvency history on file
- `sourceUrl` — direct link to the company's public profile
- `recordType: "company"`, `scrapedAt`

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `searchCompanies` | `searchCompanies` or `getCompanyDetail` |
| `apiKey` | string | – | Free Companies House API key (required — see Authentication) |
| `searchQuery` | string | – | Company name or keywords (mode=searchCompanies) |
| `companyNumbers` | array | – | UK registration numbers to fetch (mode=getCompanyDetail) |
| `companyType` | string | – (all types) | `ltd` / `plc` / `llp` / `partnership` / `sole-trader` |
| `companyStatus` | string | – (all statuses) | `active` / `dissolved` / `liquidation` / `administration` |
| `maxItems` | integer | `10` | Maximum number of company records to return (1–500) |

### Example: active technology companies

```json
{
  "mode": "searchCompanies",
  "apiKey": "your-api-key-here",
  "searchQuery": "technology",
  "companyType": "ltd",
  "companyStatus": "active",
  "maxItems": 50
}
```

### Example: look up specific registration numbers

```json
{
  "mode": "getCompanyDetail",
  "apiKey": "your-api-key-here",
  "companyNumbers": ["01234567", "09876543"]
}
```

### Example: companies in liquidation matching a keyword

```json
{
  "mode": "searchCompanies",
  "apiKey": "your-api-key-here",
  "searchQuery": "retail",
  "companyStatus": "liquidation",
  "maxItems": 100
}
```

## Use cases

- **Due diligence** — verify a company's registration status, address and filing history before doing business with it
- **Credit and risk screening** — flag companies in liquidation, administration, or with insolvency history
- **Lead list building** — pull active companies in a sector by searching a keyword with type and status filters
- **Compliance monitoring** — track upcoming accounts and confirmation-statement deadlines for a portfolio of companies
- **Market research** — profile the registered companies within an industry by SIC code

## FAQ

**Is an API key really required?**
Yes — Companies House requires an API key on every request. It's free, and registration only takes a few minutes at the link above.

**What happens if I don't provide an API key?**
The run completes successfully with an empty dataset and a status message explaining how to get a free key, instead of crashing.

**What are SIC codes?**
Standard Industrial Classification codes — the UK government's system for classifying a company's main business activity. A company can have more than one.

**How do I look up a company by its registration number instead of searching by name?**
Switch `mode` to `getCompanyDetail` and list the registration numbers in `companyNumbers`.

**Can I filter to only active companies, or only dissolved ones?**
Yes — set `companyStatus` to `active`, `dissolved`, `liquidation`, or `administration`.

**Which company types are covered?**
The `companyType` filter covers `ltd`, `plc`, `llp`, `partnership` and `sole-trader`; Companies House itself tracks additional, less common legal forms that may appear in results when no type filter is set.

**How current is the data?**
Companies House updates its register as companies file documents, and every run queries it live, so results reflect the current public record.

**Is this affiliated with Companies House or the UK government?**
No. This is an independent, third-party actor built on Companies House's own public API; it is not affiliated with or endorsed by Companies House or the UK government.
