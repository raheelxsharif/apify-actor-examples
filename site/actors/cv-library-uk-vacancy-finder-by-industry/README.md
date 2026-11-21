# CV-Library UK Vacancy Finder by Industry & Location

Pull live UK vacancies from CV-Library.co.uk, searching by job title and city or browsing whole industry sectors, and get back job title, company, salary, location, type and a direct apply link for each opening.

## What this actor does

- Searches CV-Library by keyword and UK location, or browses an entire industry sector without a keyword
- Narrows results to a specific employment type: permanent, contract, temporary, part-time or full-time
- Pages through CV-Library's results automatically up to the number of jobs you ask for
- Leaves any field CV-Library didn't publish for a listing out of the record rather than sending it empty

## Output per job

- `jobId` — CV-Library's unique listing ID
- `jobTitle` — the role title
- `companyName` — hiring company, when CV-Library discloses it
- `location` — city or region as listed
- `salary` — salary range or rate, when stated
- `jobType` — employment type (Permanent, Contract, Temporary, etc.)
- `postedDate` — date the listing went live
- `jobUrl` — direct link to the job on CV-Library
- `recordType` — always `"job"`
- `scrapedAt` — when this actor fetched the record

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string (select) | `searchJobs` | `searchJobs` (keyword + location) or `searchByIndustry` (browse a sector) |
| `query` | string | `software developer` | Job title or keyword for `searchJobs` |
| `location` | string | `London` | UK city or region for `searchJobs`; leave blank to search the whole country. Not used by `searchByIndustry` |
| `industry` | string (select) | – | Sector to browse for `searchByIndustry` — e.g. `it`, `healthcare`, `construction`, `legal` |
| `jobType` | string (select) | `` (Any) | `full-time`, `part-time`, `contract`, `temporary`, or `permanent` |
| `maxItems` | integer | `50` | Hard cap on returned job records (1-500) |

### Example: permanent developer roles in London

```json
{
  "mode": "searchJobs",
  "query": "software developer",
  "location": "London",
  "jobType": "permanent",
  "maxItems": 50
}
```

### Example: contract Python roles, anywhere in the UK

```json
{
  "mode": "searchJobs",
  "query": "Python developer",
  "jobType": "contract",
  "maxItems": 100
}
```

### Example: browse the healthcare sector

```json
{
  "mode": "searchByIndustry",
  "industry": "healthcare",
  "maxItems": 50
}
```

## Use cases

- Building a live feed of UK vacancies for a job board or alert service
- Tracking salary ranges advertised for a given role across different UK cities
- Measuring hiring volume in a sector over time for market or workforce research
- Watching a competitor's job postings for hiring-activity signals
- Sourcing candidates by pulling a fresh list of open roles in a target industry

## FAQ

**What is CV-Library?**
CV-Library is a UK job board that lists vacancies across most industries and regions, searchable by keyword or browsable by sector.

**Can I search without an industry or a keyword?**
You need one or the other: set `mode` to `searchJobs` with a `query`, or to `searchByIndustry` with an `industry` to browse a whole sector.

**Can I get jobs from the whole UK instead of one city?**
Yes — leave `location` empty on `searchJobs` to search nationwide.

**Why do some jobs have no `companyName` or `salary`?**
Those details are optional when an employer posts a listing on CV-Library; if a listing doesn't include them, this actor leaves the field out rather than returning an empty value.

**What employment types can I filter by?**
Permanent, Contract, Temporary, Part-Time, and Full-Time — or leave `jobType` blank to get every type.

**How current are the listings?**
Each record carries the `postedDate` from CV-Library plus a `scrapedAt` timestamp for the moment this actor read it; CV-Library itself adds new listings continuously.

**Is this affiliated with CV-Library?**
No. This is an independent actor that reads CV-Library's publicly available search and listing pages.
