# Totaljobs UK Vacancy Finder by Keyword, Area & Pay

Totaljobs carries tens of thousands of live UK vacancies, from warehouse shifts in Leeds to NHS roles in Manchester and contract work in London. This actor turns those public listing pages into structured records you can filter, chart or load into a spreadsheet: full descriptions, salary bands split into numbers and pay periods, exact posting and expiry dates, employer details, remote flags and apply links.

## What this actor does

- **Three ways in:** search a keyword and area, browse every vacancy in an area, or paste Totaljobs search/company/job URLs.
- **Filters that the site honours:** date posted, work-from-home only or excluded, five employment types, and newest-first or relevance ordering.
- **Salary band filtering:** enter a yearly range and the actor keeps only postings whose advertised pay overlaps it, converting hourly and daily rates to a yearly figure.
- **Rich per-job detail:** the actor can open each vacancy to collect the complete description, exact ISO dates, structured salary, employer profile and map coordinates.
- **No duplicates:** recurring jobs, cross-posted listings and overlapping search slices are de-duplicated by Totaljobs job ID.
- **Empty fields are omitted**, never returned as `null` or a blank string.

## Output per job

- `jobId` — Totaljobs' numeric listing ID
- `title` — job title as posted
- `company` — employer name
- `companyId`, `companyUrl`, `companyLogoUrl`, `companyJobsCount`, `companySectors[]` — employer profile data when published
- `companyIndustries[]`, `companyEmployees`, `companyFounded`, `companyAddress`, `companyHeroImageUrl` — extras from the employer's company profile when it has one
- `location` — location text as shown (often includes the postcode area)
- `locationPostcode`, `locationCountry` — parsed from the listing
- `locations[]` — one entry per location for multi-site vacancies (`locality`, `region`, `country`, `postalCode`, `latitude`, `longitude`)
- `locationLatitude`, `locationLongitude` — coordinates for the primary location when the employer supplies them
- `salary` — salary exactly as advertised (e.g. `£45,000 - £55,000 per annum`)
- `salaryMin`, `salaryMax`, `salaryCurrency`, `salaryPeriod` — parsed pay band (`year`, `month`, `week`, `day` or `hour`)
- `salaryAnnualMin`, `salaryAnnualMax` — annualised equivalent used by the salary filter
- `description` — full plain-text description when job pages are opened
- `descriptionHtml` — the employer's original formatting
- `postedDate` — ISO 8601 posting date when job pages are opened; the relative label (`3 days ago`) otherwise
- `postedDateText` — the relative label shown on the search results
- `validThrough` — ISO 8601 expiry date
- `employmentType` — e.g. `FULL_TIME`, `CONTRACTOR`, `REMOTE_WORKING` (multiple values joined with commas)
- `contractType` — the site's own label, e.g. `Permanent Contract; On-site`
- `workType` — `On-site`, `Hybrid` or `Remote` when the site states it
- `isRemote` — `true` for telecommute/home-working vacancies
- `applicantLocationRequirements` — country the employer accepts applicants from, for remote roles
- `industry` — Totaljobs industry classification
- `directApply` — whether the employer accepts applications on Totaljobs
- `applyType` — `internal` or `external`
- `applyUrl`, `applicationUrl` — link to start the application for this vacancy
- `jobUrl` — the vacancy page on Totaljobs
- `relatedJobs[]` — up to 10 similar vacancies (`jobId`, `title`, `url`)
- `labels[]` — badges shown on the results page (e.g. `FEATURED`, `PREMIUM`)
- `searchUrl`, `sourceKeyword`, `sourceLocation` — which search produced the row
- `firstSeenAt` — ISO timestamp when `onlyNewJobs` first returned this vacancy
- `recordType` — always `job` · `scrapedAt` — ISO timestamp

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `search` | `search` (keyword + area), `browseLocation` (whole area), `byUrls` (paste URLs) |
| `keywords` | array | – | One or more job titles or search terms (mode=search) |
| `location` | string | – | UK town, city, county or postcode; empty = UK-wide |
| `postedWithin` | string | – | `1`, `3`, `7` or `14` days (site filter) |
| `workFromHome` | string | – | `only` or `exclude` (site filter) |
| `jobType` | string | – | `permanent`, `contract`, `temporary`, `part-time`, `work-from-home` |
| `salaryMin` | integer | – | Minimum annual pay in GBP (converted for hourly/daily rates) |
| `salaryMax` | integer | – | Maximum annual pay in GBP |
| `includeDetails` | boolean | `true` | Open each job page for full descriptions, exact dates, salary bands and apply links |
| `onlyNewJobs` | boolean | `false` | Return only jobs not seen in the previous run of the same search (the first run seeds the baseline) |
| `maxItems` | integer | `100` | Maximum jobs to save (1–5,000) |
| `urls` | array | – | Totaljobs search, company or job URLs (mode=byUrls) |

Tasks saved before this revamp keep working: the old `mode` values `searchJobs` / `searchByLocation` / `searchBySalary` and the old job types `full-time` / `internship` / `apprenticeship` are accepted. The last three types are ignored because Totaljobs no longer offers them as filters.

### Example: nursing jobs across Manchester, newest first

```json
{
  "mode": "search",
  "keywords": ["registered nurse"],
  "location": "Manchester",
  "postedWithin": "7",
  "maxItems": 200
}
```

### Example: every permanent vacancy in Leeds within the last day

```json
{
  "mode": "browseLocation",
  "location": "Leeds",
  "postedWithin": "1",
  "jobType": "permanent",
  "maxItems": 100
}
```

### Example: remote data roles paying at least £55,000

```json
{
  "mode": "search",
  "keywords": ["data analyst"],
  "workFromHome": "only",
  "salaryMin": 55000,
  "includeDetails": true,
  "maxItems": 150
}
```

### Example: paste a mixed set of Totaljobs URLs

```json
{
  "mode": "byUrls",
  "urls": [
    "https://www.totaljobs.com/jobs/software-developer/in-edinburgh",
    "https://www.totaljobs.com/jobs/it-online-learning?cmpId=811550&cmp=1",
    "https://www.totaljobs.com/job/software-developer/sanderson-government-and-defence-job107993953"
  ],
  "includeDetails": true
}
```

## Use cases

- **Job seekers** — collect every matching vacancy in one list with full descriptions and pay bands, without browsing page by page.
- **Salary benchmarking** — compare advertised ranges for a role across London, Manchester, Edinburgh and other UK regions.
- **Recruitment agencies** — monitor which employers are hiring, in which sectors, and how recently each vacancy was posted.
- **Labour-market research** — measure remote-work share, contract vs permanent mix, and posting volume by area.
- **Sales and lead generation** — identify companies actively hiring in a target industry and load their profile links into a CRM.

## FAQ

**How many jobs can one run return?**
Set `maxItems` up to 5,000. Totaljobs serves at most 100 vacancies per individual search query, so for larger runs the actor automatically splits the search across the site's own location, work-from-home, job-type and skill facets, de-duplicating as it goes. In practice a broad query reaches roughly **1,000–1,400** distinct vacancies depending on the facets and related roles Totaljobs serves for that request (for "software developer" in London the site listed 2,692; runs reached 1,022, 1,200 and 1,391 unique). The actor splits by region, sub-area, work-from-home, job type, skills and related job titles; beyond the reachable set the slices repeat, so narrow the filters or split the query by area for deeper coverage.

**Why are some jobs missing salary numbers?**
Many UK employers advertise "Competitive" or omit pay entirely. The actor never invents a figure: the `salary` text is kept exactly as published, and numeric fields appear only when a real amount is shown.

**Are date filters exact?**
Yes. When you choose a date window, every returned job is checked against its posting date, so results stay inside the window even on pages where Totaljobs' own date control is unavailable.

**How does the salary filter treat hourly and daily rates?**
Hourly pay is annualised over 2,080 hours and daily rates over 220 working days. A job is kept when its annualised range overlaps your `salaryMin`/`salaryMax` window. Jobs with no published pay cannot be verified, so they are skipped while a salary filter is active.

**What is the difference between `applyUrl` and `jobUrl`?**
`jobUrl` is the public vacancy page. `applyUrl` is the application entry point for that vacancy — for jobs with external application systems, Totaljobs asks you to sign in before forwarding you to the employer's site.

**How do I get only the jobs posted since my last run?**
Set `onlyNewJobs` to `true` and keep the rest of the search identical. The actor remembers the job IDs it returned for that search, so the next run returns only unseen vacancies. The first run returns everything and becomes the baseline.

**Can I sort by relevance?**
No. Totaljobs only serves its newest-first order to anonymous requests; the relevance option depends on a signed-in session and cannot be combined with the filters. The actor always returns newest-first results.

**Why do some records lack coordinates or a company logo?**
Only what the employer and Totaljobs publish is returned. Listings without a mapped address or logo simply omit those fields.

**Does the actor need cookies, logins or a proxy setup?**
No. It uses only public pages and chooses a working UK connection automatically. Just fill in the input and run.

**Is this affiliated with Totaljobs?**
No. This is an independent tool that reads publicly available Totaljobs pages. It is not affiliated with, endorsed by, or sponsored by Totaljobs or StepStone.
