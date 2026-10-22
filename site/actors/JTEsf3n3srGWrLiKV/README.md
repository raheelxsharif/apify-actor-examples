# Florida Sunbiz Registry Lookup by Name, Agent & FEI

Look up Florida business entities the way Sunbiz itself is organized — by entity name, officer, registered agent, FEI/EIN, document number, ZIP code, street address, or trademark — and get back status, registered agent, addresses, officers, annual report history, and links to filed documents for every matching entity. No login required.

## What this actor does

- **Nine search types:** entity name, officer/registered agent name, registered agent name only, FEI/EIN number, exact document number, ZIP code, street address, trademark name, trademark owner name
- **Full detail enrichment:** every matched entity's detail page is fetched to populate registered agent, addresses, officers, annual reports, and document links
- **Status filter:** narrow results to active-only or inactive-only entities
- **Filed document links:** direct PDF links to registration statements, annual reports, amendments, and mergers where available
- **Empty fields are omitted** — a record only contains fields Sunbiz actually published for that entity

## Output per business entity

- `entityName` — official registered name
- `entityType` — e.g. `Florida Profit Corporation`, `Florida Limited Liability Company`
- `documentNumber` — Sunbiz filing document number
- `feiEinNumber` — Federal Employer Identification Number, when on file
- `filingDate` — original date filed
- `state` — state of formation (`FL` for domestic entities, home state for foreign entities)
- `status` — `ACTIVE`, `INACTIVE`, or other Sunbiz status text
- `lastEvent`, `lastEventFileDate`, `eventEffectiveDate` — most recent filing event
- `principalAddress`, `principalAddressChanged`
- `mailingAddress`, `mailingAddressChanged`
- `registeredAgentName`, `registeredAgentAddress`, `registeredAgentNameChanged`, `registeredAgentAddressChanged`
- `officers[]` — `{ title, name, address }` for each officer/director on file
- `annualReports[]` — `{ year, filedDate }` for each annual report on file
- `filedDocuments[]` — `{ label, pdfUrl? }` filing history (registration statements, annual reports, amendments, mergers) with dates. `pdfUrl` is included only when Sunbiz's document-image endpoint is reachable for that entity at scrape time; otherwise the entry still lists `label` (what was filed and when) with no link.
- `sourceUrl` — canonical Sunbiz detail-page URL
- `searchType`, `searchTerm` — the query that produced this record
- `recordType: "businessEntity"`, `scrapedAt`

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `searchType` | string (select) | `entityName` | `entityName` / `officerOrRegisteredAgent` / `registeredAgentName` / `feiEinNumber` / `documentNumber` / `zipCode` / `address` / `trademarkName` / `trademarkOwnerName` |
| `searchTerm` | string | `publix` | The value to search for |
| `entityStatus` | string (select) | `any` | `any` / `active` / `inactive` |
| `maxItems` | integer | `20` | Hard cap on emitted records (1–200). Sunbiz lists results 20 per page; the actor automatically walks Sunbiz's "Next List" pagination to satisfy any `maxItems` above 20. |
| `proxyConfiguration` | object | AUTO | Only used as an automatic fallback if Sunbiz blocks a direct request |

### Example: search by entity name

```json
{
  "searchType": "entityName",
  "searchTerm": "publix super markets",
  "maxItems": 10
}
```

### Example: search by FEI/EIN number

```json
{
  "searchType": "feiEinNumber",
  "searchTerm": "590324412",
  "maxItems": 5
}
```

### Example: exact document number lookup

```json
{
  "searchType": "documentNumber",
  "searchTerm": "112252"
}
```

### Example: active entities only, by ZIP code

```json
{
  "searchType": "zipCode",
  "searchTerm": "33811",
  "entityStatus": "active",
  "maxItems": 25
}
```

## Use cases

- **Due diligence** — verify a Florida vendor or partner's registration status and registered agent before signing a contract
- **Compliance / KYC** — confirm a business's FEI/EIN and filing history
- **Registered agent monitoring** — track which entities a registered agent represents
- **Sales / lead generation** — pull officer and address data for Florida businesses in a given area
- **Legal research** — locate an entity by document number and review its full filing history and annual reports

## FAQ

**What is the data source?**
The actor scrapes the public search portal at search.sunbiz.org, operated by the Florida Department of State, Division of Corporations. It is not affiliated with the State of Florida.

**Why does a name search sometimes return entities that don't contain my search term?**
Sunbiz's name-type searches (entity name, officer/agent name, registered agent name) perform a starts-with / nearest-alphabetical match against its sorted index — identical to the behavior of the official search UI. If your exact term has no match, the closest alphabetical neighbors are returned instead of zero results. The same "browse from this point" behavior applies to `feiEinNumber`, `zipCode`, and `address` searches too — Sunbiz's FEI/ZIP/address indexes are ordered lists, not exact-match lookups, so a FEI/EIN with only one real filing on it is padded out with the next nearest FEI numbers in the index (verified live against search.sunbiz.org itself, not an actor artifact).

**Does `documentNumber` search support partial matches?**
No — `documentNumber` is the only search type that is a true exact-match lookup (it's Sunbiz's own record primary key, so it resolves straight to the one matching detail page). A near-miss returns a "Document Not Found" status message rather than nearest neighbors. Every other search type (`entityName`, `officerOrRegisteredAgent`, `registeredAgentName`, `feiEinNumber`, `zipCode`, `address`) can return alphabetically/numerically-nearest neighbors when there's no further exact match.

**Why does `entityType` sometimes say `Trademark` instead of a corporation type?**
Sunbiz's name-based indexes (entity name, officer/agent, registered agent, ZIP, address) are shared across all filing types — corporations, LLCs, partnerships, and trademarks alike. A search that alphabetically neighbors a trademark filing will include it in results with `entityType: "Trademark"` and trademark-specific fields (owner-oriented, no registered agent/officers). This mirrors the official Sunbiz site exactly.

**How current is the data?**
Sunbiz serves live data directly from the state's filing system, so results reflect the registry at the time the actor runs.

**Are FEI/EIN numbers always available?**
No — many entities, especially older ones, have no FEI/EIN on file (or list a placeholder). The field is only included when Sunbiz has a real value.

**Why does a `zipCode` or `address` search occasionally return an entity whose principal address doesn't match?**
Sunbiz indexes entities by ZIP/address across principal, mailing, AND registered-agent addresses — a match on any one of those fields is included, matching the official site's own behavior.

**Why do some entities have no registered agent or officers listed?**
Sole proprietorships, dissolved entities, and some older filings genuinely have no registered agent or officer data on file with the state. The actor reports exactly what Sunbiz publishes.

**Why do some `filedDocuments` entries have no `pdfUrl`?**
Sunbiz's scanned-document endpoint blocks automated access independently of its search pages. The actor always reports what was filed and when (`label`); it only adds a working `pdfUrl` when the document image is actually reachable at scrape time, so you never get a dead link.

**Does this actor need a login, cookies, or proxy?**
No. Sunbiz's search portal is fully public. A proxy is only used automatically as a fallback if the source starts blocking direct requests.

**What do `trademarkName` / `trademarkOwnerName` searches return?**
The same detail-page fields as any other search (`entityType: "Trademark"`, `documentNumber`, `filingDate`, `status`, `lastEvent`, etc.) since trademark filings share Sunbiz's general detail-page layout. Trademarks have no registered agent or officers on file, so those fields are simply omitted; the owner name/address (shown under "Owners" on Sunbiz's own trademark detail page) is not currently parsed into a dedicated field.
