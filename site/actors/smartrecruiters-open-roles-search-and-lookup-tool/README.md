# SmartRecruiters Open Roles Search & Lookup Tool

Pull job postings from SmartRecruiters, the applicant-tracking platform thousands of companies use for their public job boards. Fetch every open role from a specific company by its SmartRecruiters ID, or search by keyword, location, experience level, and employment type across a curated set of companies that use SmartRecruiters.

## What this actor does

- **Two modes:** fetch all postings for one or more named companies, or search by keyword across a curated set of well-known SmartRecruiters companies
- **Location filters:** narrow by ISO country code and city name
- **Role filters:** experience level and employment type
- **Rich taxonomy per job:** department, industry, job function, and company-defined custom fields, each with both an ID and a human-readable label
- **Empty fields are omitted** — a field only appears when SmartRecruiters' own posting actually has a value for it

## Output per job

- `id`, `uuid` — SmartRecruiters posting identifiers
- `title` — job title
- `refNumber` — the company's internal reference number for the posting, when set
- `jobAdLanguage` — language code of the job ad
- `location` — object with `city`, `country`, `region`, `address`, `postalCode`, `fullLocation`, `remote`, `hybrid`, `latitude`, `longitude` — each included only when the posting sets it
- `department`, `industry`, `experienceLevel`, `typeOfEmployment`, `function` — each an object with `id` and `label`
- `customField` — company-defined custom fields, each with `fieldId`, `fieldLabel`, and its value (`value`, `valueId`, or `valueLabel` depending on field type)
- `creator` — object with `name` and `avatarUrl` for who posted the role, when published
- `createdOn`, `updatedOn`, `releasedDate` — ISO 8601 timestamps for the posting's lifecycle
- `language` — object with `code` and `label` for the job ad's language, when distinct from `jobAdLanguage`
- `visibility` — the posting's visibility setting, when published
- `companyIdentifier` — the company's SmartRecruiters slug
- `companyName` — human-readable company name
- `postingUrl`, `sourceUrl` — the public job posting URL
- `recordType` — always `job`
- `scrapedAt` — when the record was fetched

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `searchJobs` | `getJobs` (specific companies) or `searchJobs` (keyword search across a curated set of companies) |
| `companyIds` | array | – | SmartRecruiters company identifiers (required for `getJobs`) |
| `query` | string | – | Keyword to search for across the companies searched in `searchJobs` mode |
| `country` | string | – | ISO 3166 two-letter country code, e.g. `us`, `gb`, `de` |
| `city` | string | – | City name filter, e.g. `London`, `New York` |
| `experienceLevel` | string | – | `no_experience`, `junior`, `mid`, `senior`, `lead`, or `executive` |
| `typeOfEmployment` | string | – | `permanent`, `temporary`, `contract`, `part_time`, or `internship` |
| `maxItems` | integer | `100` | Maximum number of job postings to return (1–10,000) |

### Example: fetch all open roles at one company

```json
{
  "mode": "getJobs",
  "companyIds": ["GOOG"],
  "maxItems": 50
}
```

### Example: keyword search with filters

```json
{
  "mode": "searchJobs",
  "query": "product manager",
  "country": "us",
  "experienceLevel": "senior",
  "maxItems": 100
}
```

### Example: multiple companies at once

```json
{
  "mode": "getJobs",
  "companyIds": ["AveryDennison", "Microsoft"],
  "typeOfEmployment": "permanent"
}
```

## Use cases

- **Talent sourcing** — pull every open role at a target company for outreach or research
- **Market mapping** — search by keyword across SmartRecruiters companies to see who's hiring for a role
- **Hiring trend tracking** — monitor `createdOn` dates to see how fast a company is opening new roles
- **Job board aggregation** — feed SmartRecruiters-powered postings into a broader jobs dataset
- **Competitive intelligence** — compare department and experience-level mix across competing employers

## FAQ

**Is this affiliated with SmartRecruiters?**
No. This is an independent, third-party actor that reads SmartRecruiters' publicly posted job listings and is not affiliated with or endorsed by SmartRecruiters.

**How do I find a company's SmartRecruiters identifier?**
Check the company's job board URL. If postings live at `https://jobs.smartrecruiters.com/GOOG/...`, the identifier is `GOOG`. The company name in its PascalCase or slug form (e.g. `AveryDennison`) usually works too.

**Do I need a login or API key?**
No. SmartRecruiters' job postings are public, and the actor runs without any authentication.

**Can I fetch roles from more than one company in a single run?**
Yes — list multiple identifiers in `companyIds` under `getJobs` mode, and the actor returns postings from all of them.

**How do I find remote roles?**
Use `searchJobs` with a keyword like `remote software engineer`, since `location.remote` reflects what the posting itself declares rather than being a standalone filter.

**Why are `customField` entries different shapes between postings?**
SmartRecruiters lets each company define its own custom fields, and a field's value can be free text (`value`) or a selected option (`valueId`/`valueLabel`) depending on how that company configured it.

**Why might `department`, `industry`, or `function` be missing from a job?**
Those are optional taxonomy fields. If a company didn't set one when posting the role, it's left out of the record rather than sent as an empty object.

**How many postings can one run return?**
Up to `maxItems`, capped at 10,000. For a company with an unusually large board, running multiple times with different filters can help split up the work.

**Is the data current?**
Yes — each run reads SmartRecruiters' live postings directly, so results reflect what's published at run time.
