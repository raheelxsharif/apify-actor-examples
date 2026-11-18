# Washington L&I Contractor & Tradesperson License Scraper

Scrape **Washington State Department of Labor & Industries (L&I)** contractor, electrician, plumber, elevator, and asbestos-worker license records from the official "Verify a Contractor, Tradesperson or Business" public registry. Search by business/individual name, license or registration number, UBI (Unified Business Identifier), or workers' comp account ID. Optionally enrich every result with the full license detail record — address, surety bonds, liability insurance, business owners, associated licenses, and safety/contractor violations. No login, no API key, no proxy required.

## What this actor does

- **Four search modes:** by name, by license/registration/certificate number, by UBI number, by workers' comp account ID
- **20 license categories covered:** general/specialty construction contractors, electrical (contractor, administrator, electrician, master electrician, trainee), plumbing (contractor, plumber, plumber trainee, medical gas), asbestos (contractor, supervisor, worker), elevator (contractor, mechanic, primary contact), and manufactured home installers
- **Optional detail enrichment:** attach full business detail — surety bonds, liability insurance, business owners/officers, associated licenses, safety and license violations, DOSH inspections
- **Filters:** license category, license status (active/inactive), city, zip code, and violations-only
- **Empty fields are omitted** — every record only contains data L&I actually returned for that license

## Data source

This actor reads Washington State L&I's own public "Verify a Contractor, Tradesperson or Business" lookup tool — the same results the official website's search page shows. No login, API key, or proxy is required.

## Output per license record

- `licenseId` — the L&I license, registration, or certificate number
- `ubi` — Washington Unified Business Identifier, when on file
- `businessName` — business or individual licensee name
- `contractorType` / `licenseType` — e.g. `Electrician`, `Asbestos Worker`, `General/Specialty Contractor`
- `contractorGroup` — top-level category (e.g. `Electrical`, `Plumber`, `Asbestos`)
- `asbestosGroup` — asbestos-program sub-classification, when the license is asbestos-related
- `status` — `Active`, `Inactive`, `Expired`, `Suspended`, `Out of Business`, `Re-Licensed`, or `Archived`
- `statusDescription` — the raw L&I status text (e.g. `EXPIRED`, `ACTIVE`, `SUSPENDED`) — detail mode only
- `employerDisplayStatus` — L&I's workers'-comp employer-account display status, when on file
- `city`, `state`, `zipCode`, `county`, `address1`, `address2`, `phoneNumber` — detail mode only for address/phone
- `businessType` — e.g. `Limited Liability Company`, `Individual`, `Corporation`
- `parentCompany` — parent/umbrella business, when on file
- `effectiveDate`, `expirationDate`, `suspendDate` — ISO `YYYY-MM-DD`
- `specialtyCode1`, `specialtyName1`, `specialtyText1` and `specialtyCode2`/`specialtyName2`/`specialtyText2` — construction contractor specialty classification
- `contractorTypeDescription` — plain-language description of the license type
- `licenseTypeCode` — L&I's internal short code for the license type (e.g. `CC`) — detail mode only
- `relationType` — the licensee's relation type on the record (e.g. `Business`, `Individual`) — detail mode only
- `hasSafetyViolation`, `hasContractorViolation` — boolean flags
- `hasInactiveWorkersCompAccounts` — boolean
- `associatedLicenses[]` — other licenses/registrations tied to the same business or person
- `businessOwners[]` — owners/officers/partners, with role and start/end dates
- `bonds[]` — surety bond company, account number, amount, effective/expiration dates
- `insurances[]` — liability insurance firm, account ID, amount, effective/expiration dates
- `savingsAccounts[]` — savings account assignment in lieu of bond, when on file
- `violations[]`, `summons[]`, `warrants[]`, `debarments[]`, `strikes[]` — enforcement history, when on file
- `lapses[]` — license lapse/reinstatement history
- `courses[]`, `affidavits[]`, `codes[]` — continuing education and compliance records, when on file
- `doshInspections[]` — Washington DOSH workplace safety inspection history, when on file
- `employerBusinessDetails` — workers'-comp employer business record (legal name, business ID, address), when on file
- `contractorTrackingUrl`, `contractorRegistrationUrl` — related L&I portal links
- `sourceUrl` — canonical L&I Verify detail page URL for the license
- `recordType: "license"`, `scrapedAt`

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `byName` | `byName` / `byLicenseId` / `byUbi` / `byAccountId` |
| `name` | string | `smith` | Business name or individual last name (mode=byName); min 2 characters |
| `licenseId` | string | – | License/registration/certificate number, exact or prefix (mode=byLicenseId) |
| `ubi` | string | – | 9-digit Washington UBI number (mode=byUbi) |
| `accountId` | string | – | Workers' comp account ID, e.g. `123,456-01` (mode=byAccountId) |
| `city` | string | – | Restrict results to a single Washington city (case-insensitive, applied server-side by L&I) |
| `zipCode` | string | – | Restrict results to a single 5-digit Washington zip code (applied server-side by L&I; ignored if `city` is also set) |
| `licenseCategory` | string | – | Restrict results to one of 20 license categories |
| `statusFilter` | string | – | `active` / `inactive` / `expired` / `suspended` / `out of business` / `re-licensed` / `archived` |
| `violationsOnly` | boolean | `false` | Only emit records with a safety and/or contractor violation flagged |
| `sortOrder` | string | `desc` | `desc` / `asc` — L&I's own relevance ranking |
| `fetchDetails` | boolean | `true` | Enrich each result with the full detail record (bonds, insurance, owners, violations, etc.) |
| `maxItems` | integer | `10` | Hard cap on emitted license records (1–500) |

### Example: search by business name with detail enrichment

```json
{
  "mode": "byName",
  "name": "smith",
  "fetchDetails": true,
  "maxItems": 20
}
```

### Example: look up an exact license number

```json
{
  "mode": "byLicenseId",
  "licenseId": "ABCAR824BB",
  "fetchDetails": true
}
```

### Example: search by UBI number, active licenses only

```json
{
  "mode": "byUbi",
  "ubi": "604164913",
  "statusFilter": "active"
}
```

### Example: browse all licensed electricians without detail hydration (faster)

```json
{
  "mode": "byName",
  "name": "construction",
  "licenseCategory": "Electrician",
  "fetchDetails": false,
  "maxItems": 100
}
```

### Example: search by name restricted to one city

```json
{
  "mode": "byName",
  "name": "construction",
  "city": "Spokane",
  "maxItems": 25
}
```

### Example: compliance check for licenses with flagged violations

```json
{
  "mode": "byName",
  "name": "construction",
  "violationsOnly": true,
  "maxItems": 25
}
```

## Use cases

- **Due diligence** — verify a contractor's license, bond, and insurance status before hiring
- **Compliance monitoring** — track license status and violation history for vendors and subcontractors
- **Lead generation** — build lists of licensed contractors/tradespeople by category or region
- **Insurance & surety underwriting** — cross-check bond and insurance coverage on file with L&I
- **Legal & background research** — pull violation, debarment, and lapse history tied to a business or UBI

## FAQ

**What is the data source?**
This actor uses Washington State Department of Labor & Industries' own public "Verify a Contractor, Tradesperson or Business" lookup tool at `secure.lni.wa.gov/verify`. It is a genuine public REST API with no authentication requirement — the same data the L&I website itself displays.

**Is this affiliated with Washington State L&I?**
No. This is an independent, third-party actor that reads L&I's public verification data. It is not affiliated with or endorsed by the Washington State Department of Labor & Industries.

**Why does `fetchDetails` matter?**
Search results alone give you the license number, business name, category, and basic status. Turning on `fetchDetails` (the default) fetches the full record for each result — address, bonds, insurance, business owners, and violation history — at the cost of one extra request per record. Turn it off for faster, lighter-weight bulk name/category browsing.

**Why are some fields missing on certain licenses?**
L&I only returns the fields it has on file for a given license. Bonds, insurance, and violation history are commonly empty for license types that don't require them (e.g. individual trade certifications like Electrician or Asbestos Worker don't carry bond/insurance records the way business contractor licenses do). Empty fields are always omitted rather than shown as blank or null.

**What's the difference between "Construction Contractor" and the electrical/plumbing/asbestos/elevator categories?**
"Construction Contractor" is L&I's general contractor and specialty-trade contractor registration (bonded businesses). Electrical, plumbing, asbestos, and elevator categories are separate certification/licensing programs for individual tradespeople and specialty contractors in those disciplines, each with their own bonding/insurance rules.

**How fresh is the data?**
Every run queries the live L&I API directly — there is no caching layer, so results reflect L&I's registry at the moment the actor runs.

**Can I narrow results to a specific city or zip code?**
Yes. Set `city` (e.g. `Seattle`) or `zipCode` (e.g. `98121`) alongside any search `mode` — L&I's own registry applies the location filter server-side on top of your name/license/UBI/account search, the same as the location dropdown on the official Verify tool's results page. If both are set, `city` takes priority.

**Can I search by partial name?**
Yes. `name` (mode=byName) and `licenseId` (mode=byLicenseId) both support partial/prefix matching, mirroring L&I's own search behavior — the same as typing into the site's search box.

**Does this include Washington's professional licenses (real estate, cosmetology, etc.)?**
No — those are handled by a different Washington State agency (the Department of Licensing, DOL) with a separate lookup tool. This actor is scoped to L&I's contractor, tradesperson, and workers'-comp registry only.
