# Remotive Remote Job Listings Extractor

Browse or search Remotive's remote job board without clicking through category pages yourself. Remotive curates remote positions across software, design, marketing, data, and more; this actor reads its public listings and returns each job as a structured record with company, salary, tags, and a direct apply link.

## What this actor does

- **Two modes:** `browse` returns every currently listed job; `search` adds keyword matching on top
- **Filters:** category, candidate-required location, and employment type apply in both modes
- **Full job content:** plain-text description, skill tags, salary range when posted, and both the listing and apply URLs
- **Keyword matching:** `query` searches across job title, tags, and description text
- **Empty fields are omitted** — a listing without a posted salary, for example, simply has no `salary` field

## Output per job

- `id` — Remotive's unique job ID
- `title` — job title
- `companyName` — hiring company name
- `companyLogo` — company logo image URL, when provided
- `category` — job category (e.g. `Software Development`, `Design`)
- `jobType` — employment type (e.g. `Full-time`, `Contract`)
- `tags` — up to 20 skill/technology tags
- `location` — candidate-required location (e.g. `USA`, `Europe`, `Worldwide`)
- `salary` — salary range text, when the listing includes one
- `description` — plain-text job description, with HTML markup stripped
- `publicationDate` — ISO 8601 date the job was posted
- `jobUrl`, `applyUrl` — the listing page and apply link (the same URL on Remotive)
- `sourceUrl` — the listing URL the record was built from
- `remote` — always `true`, since every Remotive listing is remote
- `recordType` — always `job`
- `scrapedAt` — when the record was fetched

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `browse` | `browse` (all current jobs) or `search` (keyword + filters) |
| `query` | string | – | Keyword matched against title, tags, and description (mode=`search`) |
| `category` | string | – | Job category, e.g. `software-dev`, `design`, `marketing`. Leave blank for all categories |
| `location` | string | – | Candidate-required location substring, e.g. `USA`, `Europe`, `Worldwide` |
| `jobType` | string | – | `Full-time`, `Part-time`, `Contract`, `Freelance`, `Internship`, or `Temporary` |
| `maxItems` | integer | `100` | Maximum number of job records to return (1–10,000) |

### Example: browse all current listings

```json
{
  "mode": "browse",
  "maxItems": 50
}
```

### Example: search for a role with location and type filters

```json
{
  "mode": "search",
  "query": "python developer",
  "location": "USA",
  "jobType": "Full-time",
  "maxItems": 20
}
```

### Example: browse one category only

```json
{
  "mode": "search",
  "category": "design",
  "maxItems": 30
}
```

## Use cases

- **Remote job board aggregation** — feed Remotive's listings into a wider job-search product
- **Hiring market research** — see how many open roles exist in a category, location, or employment type
- **Candidate alerts** — run on a schedule and diff against the last run to spot newly posted roles
- **Recruiting lead sourcing** — collect company names and job categories currently hiring remotely
- **Salary benchmarking** — gather posted salary ranges across a category or role type

## FAQ

**Is this affiliated with Remotive?**
No. This is an independent, third-party actor that reads Remotive's publicly available job listings and is not affiliated with or endorsed by Remotive.

**Do I need an account or API key?**
No. Remotive's job listings used here are public, and the actor doesn't require any login or key.

**What's the difference between `browse` and `search`?**
Both modes accept `category`, `location`, and `jobType` filters. `search` additionally matches `query` against the job title, tags, and description; `browse` ignores `query` and returns every job that passes the other filters.

**Why is `salary` sometimes missing?**
Not every company posts a salary range. When a listing doesn't include one, the field is left out rather than sent as empty text.

**Can I filter by more than one category at a time?**
Not in a single run — `category` accepts one value per run. Run the actor again with a different category to cover more than one.

**Is the description HTML or plain text?**
Plain text. Any HTML markup in the original listing is stripped before the record is built.

**How fresh are the listings?**
Each run reads Remotive's current listings live, so results reflect whatever is posted at run time.

**Can I search by company name?**
Yes — put the company name in `query`; it's matched against title, tags, and description, which will catch most mentions of the company.
