# Greenhouse ATS Job Board Lookup by Company

Pull every open role from one or more company career pages built on Greenhouse — the applicant tracking system used by thousands of companies including Airbnb, Stripe, and Figma — or look up specific postings by job ID, with location, department, and apply link for each.

## What this actor does

- Fetches every open job from one or more Greenhouse company board slugs in a single run.
- Looks up specific job postings directly by company slug and job ID.
- Returns each job's location breakdown, departments, offices, employment type, and salary range when the company publishes one.
- Can optionally include the full HTML job description alongside the structured fields.
- Fields Greenhouse doesn't publish for a given posting (such as `salary` when a company doesn't disclose one) are left out of that record.

## Output per job

| Field | Description |
|---|---|
| `id` | Greenhouse job ID |
| `title` | Job title |
| `companySlug` | Greenhouse company board slug |
| `companyName` | Company display name, when published |
| `location` | Location object with `name`, `city`, `state`, `country` |
| `departments` | List of department names |
| `offices` | List of office names |
| `employmentType` | Employment type (e.g. "Full-time", "Contract") |
| `educationLevel` | Required education level, when specified |
| `salary` | Salary range object, when the company publishes one |
| `requisitionId` | Internal requisition ID |
| `updatedAt` | Last-updated timestamp from Greenhouse |
| `applyUrl` | Direct application URL |
| `absoluteUrl` | Full job listing URL on Greenhouse |
| `sourceUrl` | Constructed board URL for the job |
| `content` | Full HTML job description (only when `content` is enabled) |
| `recordType` | Always `job` |
| `scrapedAt` | ISO 8601 timestamp of when the record was scraped |

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `getJobs` | `getJobs` (all jobs from company boards) or `getJob` (specific job by ID) |
| `companySlugs` | array | – | Greenhouse company board slugs to scrape (mode=`getJobs`) |
| `companySlug` | string | – | Single company board slug (mode=`getJob`) |
| `jobIds` | array | – | Greenhouse job IDs to fetch (mode=`getJob`) |
| `content` | boolean | `false` | Include the full job description as HTML |
| `maxItems` | integer | `500` | Maximum number of job records to emit (1–10000) |

### Example: all open roles across three companies

```json
{
  "mode": "getJobs",
  "companySlugs": ["stripe", "airbnb", "shopify"],
  "maxItems": 1000
}
```

### Example: specific job postings with full description

```json
{
  "mode": "getJob",
  "companySlug": "stripe",
  "jobIds": [4567890, 4567891],
  "content": true
}
```

### Example: one company's jobs with descriptions, capped

```json
{
  "mode": "getJobs",
  "companySlugs": ["figma"],
  "content": true,
  "maxItems": 100
}
```

## Use cases

- **Job board aggregation** — build a career site or job feed from live Greenhouse data
- **Hiring trend research** — track which departments and locations a company is actively hiring for
- **Competitive intelligence** — monitor which roles competitors are trying to fill
- **Recruitment analytics** — compare job requirements and employment types across companies
- **Salary benchmarking** — collect published salary ranges where companies disclose them

## FAQ

**Do I need an API key or login?**
No — Greenhouse's public job board API is open and requires no authentication.

**How do I find a company's board slug?**
It's the identifier in the company's Greenhouse board URL, e.g. `stripe` in `boards.greenhouse.io/stripe`.

**Does this work for every company that uses Greenhouse?**
It works for any company with a public Greenhouse board. Some companies configure their board as private, in which case it isn't accessible through the public API at all.

**How do I get the full job description text?**
Set `content` to `true`; each job record will then include the full HTML description.

**How many jobs can I pull per run?**
Up to `maxItems`, capped at 10,000. Greenhouse's API returns a company's full job list in one response, with no pagination needed.

**Why is `salary` missing from some jobs?**
Not every company publishes a salary range on Greenhouse; the field is included only when the company's own posting has one.

**What happens if I give an invalid company slug?**
That slug is skipped with a logged warning, and the run continues with any other slugs you provided.

**How fresh is the data?**
As fresh as Greenhouse's own API — each job carries an `updatedAt` timestamp straight from the source.

**Is this affiliated with Greenhouse Software?**
No. This is an independent, third-party actor that reads Greenhouse's public job board data and is not operated by or affiliated with Greenhouse Software, Inc.
