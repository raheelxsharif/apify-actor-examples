# UK Companies House Profile & Filing Status Scraper

Companies House is the UK government's register of companies. This actor reads the live public register to find companies by name, legal form, status, location, SIC code or date range, and then returns detailed profiles, officer appointments, or filing history.

It works out of the box with no login, cookie, proxy or API key. If you already have a free Companies House API key, add it and the actor uses the official API for structured addresses, jurisdiction, overdue flags, and faster officer/filing pages. If the key is missing or rejected, the public website path takes over automatically.

## Modes

| Mode | What it returns |
|---|---|
| `searchCompanies` | One record per matching company: number, name, status, type, incorporation/dissolution date, registered office, SIC codes |
| `getCompanyProfile` | Full profile: address, SIC descriptions, previous names, accounts and confirmation-statement dates, charge and insolvency flags |

Profiles cover ordinary UK companies plus UK establishments, overseas companies, and overseas
entities, including their `Status`, `Opened on`, and `Registered on` register dates.
| `getCompanyOfficers` | Officer appointments: name, role, status, dates, nationality, residence, occupation (with API key), date of birth, identity verification, address |
| `getFilingHistory` | Filing events: date, type, category, description, transaction ID, document link, page count |

All lookup modes accept registration numbers, profile URLs, or a mixture of both in `companyNumbers`.

## Search filters and pagination

Search combines any of:

- company name or keyword
- register status (active, dissolved, liquidation, administration, and others)
- legal form (Ltd, PLC, LLP, limited partnership, and every other type in the advanced register search)
- registered office location
- one SIC code
- incorporation and dissolution date ranges

Results are paged automatically until `maxItems` is reached. A search can return up to 5,000 records, and company-number duplicates are removed.

Officer and filing modes also page through the register and apply `maxItems` across every company in the list. `officerStatus` narrows officers to current or resigned, and `filingCategory` narrows filings to accounts, capital, charges, confirmation statements, incorporation, or officers.

## Output fields

### Company records

- `recordType` — `company`
- `companyNumber`, `companyName`
- `companyType` — register code such as `ltd`, `plc`, `llp`
- `companyTypeLabel` — human-readable legal form
- `companyStatus`, `dateOfCreation`, `dateOfCessation`
- `registeredOfficeAddress` — structured lines when available, plus `fullAddress`
- `sicCodes[]`, `sicDescriptions[]`
- `accounts` — `nextDue`, `nextMadeUpTo`, `lastMadeUpTo`, `overdue` (API key)
- `confirmationStatement` — same date fields
- `previousNames[]` — name, `from`, `to`
- `jurisdiction` — API key only
- `hasCharges`, `hasInsolvencyHistory`
- `sourceUrl`, `scrapedAt`

### Officer records

- `recordType` — `officer`
- `companyNumber`, `companyName`
- `officerName`, `officerRole`, `officerStatus`
- `appointedOn`, `appointedBefore`, `resignedOn`
- `dateOfBirth` (month precision), `nationality`, `countryOfResidence`, `occupation`
- `correspondenceAddress`, `identityVerificationStatus`
- `officerType`, `registrationNumber` for corporate officers
- `officerAppointmentsUrl`, `sourceUrl`, `scrapedAt`

### Filing records

- `recordType` — `filing`
- `companyNumber`, `companyName`
- `filingDate`, `filingType`, `category`, `description`
- `actionDate`, `transactionId`, `pages`
- `documentUrl` — direct PDF view link
- `annotations[]`, `descriptionValues` when the register provides them
- `sourceUrl`, `scrapedAt`

Empty values are omitted; fields the register does not hold for a company are never filled with placeholders.

## Input reference

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `searchCompanies` | `searchCompanies`, `getCompanyProfile`, `getCompanyOfficers`, or `getFilingHistory` |
| `apiKey` | string | – | Optional free Companies House API key; never required |
| `searchQuery` | string | – | Company name or keyword for search mode |
| `companyNumbers` | array | – | Registration numbers or profile URLs for the lookup modes |
| `companyStatus` | string | All statuses | One register status, or all statuses |
| `companyType` | string | All types | One legal form, or all types |
| `registeredOfficeAddress` | string | – | Full or partial registered-office location |
| `sicCode` | string | – | One 4–5 digit SIC code |
| `incorporatedFrom` | string | – | Earliest incorporation date, YYYY-MM-DD |
| `incorporatedTo` | string | – | Latest incorporation date, YYYY-MM-DD |
| `dissolvedFrom` | string | – | Earliest dissolution date, YYYY-MM-DD |
| `dissolvedTo` | string | – | Latest dissolution date, YYYY-MM-DD |
| `officerStatus` | string | `all` | `all`, `active`, or `resigned` |
| `filingCategory` | string | All filings | `accounts`, `capital`, `charges`, `confirmation-statement`, `incorporation`, or `officers` |
| `maxItems` | integer | `25` | Maximum records per run, 1–5000 |

## Example inputs

Search active private companies matching a keyword:

```json
{
  "mode": "searchCompanies",
  "searchQuery": "renewable energy",
  "companyStatus": "active",
  "companyType": "ltd",
  "maxItems": 50
}
```

Find companies in a location and incorporation window:

```json
{
  "mode": "searchCompanies",
  "searchQuery": "consulting",
  "registeredOfficeAddress": "Cardiff",
  "incorporatedFrom": "2015-01-01",
  "incorporatedTo": "2024-12-31",
  "maxItems": 100
}
```

Pull profiles for registration numbers or pasted URLs:

```json
{
  "mode": "getCompanyProfile",
  "companyNumbers": [
    "00445790",
    "https://find-and-update.company-information.service.gov.uk/company/05888957"
  ],
  "maxItems": 10
}
```

List current officers for a portfolio:

```json
{
  "mode": "getCompanyOfficers",
  "companyNumbers": ["00445790"],
  "officerStatus": "active",
  "maxItems": 50
}
```

Download filing history in one category:

```json
{
  "mode": "getFilingHistory",
  "companyNumbers": ["00445790"],
  "filingCategory": "accounts",
  "maxItems": 100
}
```

Include an optional key for enriched profile data:

```json
{
  "mode": "getCompanyProfile",
  "apiKey": "your-free-companies-house-key",
  "companyNumbers": ["00445790"]
}
```

## Use cases

- **Supplier and partner due diligence** — confirm a company is active, check its registered address, and review accounts and confirmation-statement deadlines.
- **Credit and risk screening** — flag liquidation, administration, insolvency history, and charges before extending credit.
- **KYC and compliance files** — collect officer names, roles, appointment dates, and identity-verification status for a company file.
- **Filing deadline monitoring** — export accounts and confirmation-statement due dates for a portfolio, then schedule the actor to refresh them.
- **Market and lead research** — build lists of active companies by legal form, location, SIC code, or incorporation year.
- **Document retrieval** — locate filing events and open the original PDF for each event.

## FAQ

**Do I need a Companies House API key?**
No. The public register pages are the default source. A free API key is optional and only adds structured address fields, jurisdiction, overdue flags, occupations, and API-native filing metadata.

**What happens if my API key is invalid?**
The actor logs the rejection, continues on the public register, and still returns records.

**How current is the data?**
Every run reads the live register, so records reflect the filings and status at run time.

**How far back does filing history go?**
The register exposes the full filing history it holds for the company. `maxItems` and `filingCategory` control how much is returned.

**Can I paste a company profile URL instead of a number?**
Yes. Lookup modes accept registration numbers and profile URLs.

**Why are some fields missing on a record?**
Companies House does not hold every field for every company — for example, a dissolved company has no next accounts due date. Missing values are omitted rather than fabricated.

**Can I get more than 5,000 search results?**
The advanced search supports up to 5,000 records per query. Narrow the query with status, type, location, SIC code, or date filters to cover a larger dataset in slices.

**Does this actor need a proxy?**
No. It reads the public register directly. The actor selects its own network path automatically.

**Is this affiliated with Companies House or the UK government?**
No. This is an independent third-party tool that reads publicly available register pages. It is not affiliated with or endorsed by Companies House or the UK government.
