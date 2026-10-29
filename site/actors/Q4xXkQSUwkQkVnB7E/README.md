# Bayt.com MENA Job Listing & Salary Scraper

Bayt.com is one of the largest job boards across the Middle East and North Africa, covering everything from Dubai tech roles to Cairo retail jobs. This actor searches that listing inventory by country, city, or keyword and returns each job with its company, salary, and full description.

## What this actor does

- Searches Bayt.com job listings from a direct search/category URL, or from a keyword and country/city combination.
- Walks listing pages up to a configurable page and result limit.
- Filters results by how recently they were posted, career level, and employment type.
- Visits each job's own page to collect salary, skills, preferred-candidate requirements, and company profile details.
- Leaves out fields a listing doesn't publish (no salary, no benefits list) rather than sending them empty.

## Output per job

- `jobId`, `url`, `applyUrl`, `isExternalJob`
- `title`
- `company`, `companyProfileUrl`, `companyLogo`
- `companySize`, `companyEmployeesSize`, `companyIndustry`, `companyFoundedYear`
- `location`, `jobType` (Remote / Hybrid / On-site)
- `postedAtRelative`, `postedAt`, `datePosted`
- `employmentType`, `careerLevel`, `experience`
- `industryId`, `industryName`, `jobFunction`
- `salary`, `minSalaryUSD`, `maxSalaryUSD` — the latter two only when the job page shows a USD range
- `vacancies` — number of open positions, when listed
- `descriptionHtml`, `descriptionText`, `skillsText`, `listingSummary`
- `preferredCandidate`, `gender`, `nationality`, `residenceCountry`, `degree`, `languages`
- `benefits` — list of benefits, when the job page has a benefits section
- `candidateStatus` — whether the role requires willingness to travel, when stated
- `applicationsCount`, `viewsCount` — interaction counts, when the job page shows them
- `type` — always `job_bayt`
- `scrapedAt`

If a search returns no matching jobs, the run still emits a single sentinel record (`type: "job_bayt_blocked"`) instead of an empty dataset, so a run always produces visible output you can inspect.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `startUrl` | string | `https://www.bayt.com/en/international/jobs/python-jobs/` | A Bayt.com search or category URL |
| `keyword` | string | – | Keyword filter used to build a search URL when `startUrl` is left at its default |
| `locationFilter` | string | – | Country or city slug (e.g. `uae`, `dubai-uae`, `saudi-arabia`, `qatar`) combined with `keyword` to scope the search |
| `resultsWanted` | integer | `3` | Maximum number of job listings to return (1-500) |
| `maxPages` | integer | `20` | Maximum number of listing pages to go through (1-100) |
| `datePosted` | string | `any` | `any`, `last_24_hours`, `last_week`, or `last_month` |
| `careerLevel` | string | `any` | `any`, `entry`, `mid_career`, `management`, `senior_executive`, or `student` |
| `employmentType` | string | `any` | `any`, `full_time`, `part_time`, `contract`, `internship`, or `temporary` |
| `proxyConfiguration` | object | `{"useApifyProxy": true}` | Apify Proxy configuration for the run |

### Example: keyword search in one country

```json
{
  "startUrl": "https://www.bayt.com/en/international/jobs/python-jobs/",
  "keyword": "sales",
  "locationFilter": "uae",
  "resultsWanted": 50
}
```

### Example: browse a city's listings directly

```json
{
  "startUrl": "https://www.bayt.com/en/dubai-uae/jobs/",
  "careerLevel": "mid_career",
  "resultsWanted": 100
}
```

### Example: a single job page

```json
{
  "startUrl": "https://www.bayt.com/en/uae/jobs/creative-tech-ai-lead-5443062/"
}
```

## Use cases

- Track open roles and salary ranges for a given title or industry across MENA countries.
- Build a recruiting lead list filtered by career level and employment type.
- Monitor newly posted jobs in a specific city using the `last_24_hours` or `last_week` filter.
- Compare required skills and candidate requirements across similar postings.
- Feed structured MENA job market data into a research or dashboard project.

## FAQ

**Is this affiliated with Bayt.com?**
No. This actor reads Bayt.com's own public job pages and has no relationship with Bayt.com.

**Do I need a Bayt.com account?**
No. Every field comes from job pages that are publicly visible without logging in.

**Can I scrape a single job instead of a search?**
Yes — put the direct job URL in `startUrl`; the actor recognizes a job page and skips pagination.

**Which countries are covered?**
Any Bayt.com country or city sub-site, such as UAE, Saudi Arabia, Qatar, Kuwait, Oman, Bahrain, Egypt, Jordan, Lebanon, or Morocco. Use the country or city slug from the Bayt.com URL (e.g. `/en/uae/jobs/`) in `locationFilter` or directly in `startUrl`.

**Why are `minSalaryUSD`/`maxSalaryUSD` missing from some jobs?**
They're only added when the job's own page shows a converted USD salary range; many postings only show a local-currency figure or no salary at all, so `salary` may be the only pay field present.

**Why do `careerLevel` and `employmentType` filter client-side?**
Those filters are applied to each job's extracted fields, so they work even on category pages where Bayt.com's own listing doesn't let you filter by them directly.

**Why did I get a `job_bayt_blocked` record instead of jobs?**
That's a sentinel record emitted when the search or category URL matched no listings, so a run always returns visible output instead of an empty dataset.

**How current are the listings?**
Each run searches Bayt.com live, so postings and salaries reflect what's listed at run time.
