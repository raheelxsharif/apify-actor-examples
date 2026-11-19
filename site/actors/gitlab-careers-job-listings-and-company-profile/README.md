# GitLab Careers Job Listings & Company Profile Extractor

Pull live job openings straight from GitLab's own public careers board, or grab a snapshot of GitLab's careers page itself. Search by keyword, filter by department or quota-coverage type, resolve exact job IDs or URLs, or fetch a lightweight company-profile record — all without opening a browser.

## What this actor does

- **Six modes:** keyword search, all open jobs, filter by department, resolve exact job IDs, resolve exact job URLs, or fetch a GitLab company profile
- **Full job detail on request:** title, location, department, requisition ID, publish/update timestamps, and (optionally) the full description text, HTML, section headings, and outbound links
- **Filters:** location substring, minimum publish date, minimum update date, department names, and quota-coverage type
- **Sorting:** by publish date, update date, or title, ascending or descending
- **Empty fields are omitted, and invalid lookups return a typed `error` record instead of failing silently**

## Output per record

The actor emits `job`, `company`, or `error` records, each marked by `recordType`.

### Job records
- `jobId` — the job's numeric ID
- `url` — direct apply URL for the posting
- `title` — job title
- `companyName` — always `GitLab`
- `location` — office or remote location string
- `language` — posting language code
- `requisitionId` — GitLab's internal requisition number
- `internalJobId` — GitLab's internal job ID, when distinct from `jobId`
- `firstPublishedAt`, `updatedAt` — ISO timestamps for when the posting first went live and was last updated
- `applicationDeadline` — ISO timestamp, when the posting has one
- `metadata` — any additional name/value fields GitLab attaches to the posting
- `quotaCoverageType` — sales quota-coverage classification, for sales roles that carry one
- `departmentNames` — department(s) the role belongs to
- `officeNames` — office(s) associated with the role
- `descriptionHtml`, `descriptionText` — the full job description, as HTML and as plain text, when `fetchFullDescription` is enabled
- `sectionHeadings` — heading text pulled from the description, when available
- `descriptionLinks` — outbound links found in the description
- `complianceTypes` — data-compliance classifications attached to the role, when present
- `scrapedAt` — when the record was fetched

### Company profile record
- `companyName` — `GitLab`
- `companyUrl` — GitLab's main site
- `careersUrl`, `benefitsUrl`, `valuesUrl`, `handbookUrl` — GitLab's careers, benefits, values, and handbook pages
- `remoteCompany` — `true` when the careers page content indicates GitLab is an all-remote company
- `companySummary` — the first substantial paragraph pulled from the careers page
- `introParagraphs` — up to 6 paragraphs of careers-page intro text
- `sectionHeadings` — up to 12 heading texts from the careers page

### Error record
- `errorType` — always `invalid_input`
- `message` — what went wrong (e.g. a required field was missing for the chosen mode)
- `mode`, `field` — which mode and input field triggered the error
- `scrapedAt` — when the record was produced

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `searchJobs` | `searchJobs`, `allJobs`, `byDepartment`, `byJobIds`, `byJobUrls`, or `companyProfile` |
| `searchQuery` | string | – | Keyword matched against title, company, location, and requisition ID (mode=`searchJobs`) |
| `departmentNames` | array | `[]` | Exact GitLab department names, required for `byDepartment` and usable as a filter in other list modes |
| `quotaCoverageTypes` | array | `[]` | Exact quota-coverage values to filter by (sales roles only) |
| `jobIds` | array | – | Exact job IDs to resolve (mode=`byJobIds`) |
| `jobUrls` | array | – | Exact job URLs to resolve (mode=`byJobUrls`) |
| `locationContains` | string | – | Case-insensitive location substring filter (e.g. `Remote`, `EMEA`, `US`) |
| `publishedFrom` | string | – | Keep jobs first published on or after this ISO date/time |
| `updatedFrom` | string | – | Keep jobs updated on or after this ISO date/time |
| `sortBy` | string | `firstPublishedDesc` | `firstPublishedDesc`, `firstPublishedAsc`, `updatedDesc`, `updatedAsc`, `titleAsc`, or `titleDesc` |
| `fetchFullDescription` | boolean | `true` | Fetch each job's detail page for description text, departments, offices, and headings |
| `maxItems` | integer | `5` | Maximum number of records to emit (1–5000) |

### Example: keyword search

```json
{
  "mode": "searchJobs",
  "searchQuery": "engineer",
  "maxItems": 10
}
```

### Example: all open jobs in one department

```json
{
  "mode": "byDepartment",
  "departmentNames": ["Data Engineering"],
  "sortBy": "firstPublishedDesc",
  "maxItems": 50
}
```

### Example: resolve specific job URLs

```json
{
  "mode": "byJobUrls",
  "jobUrls": ["https://job-boards.greenhouse.io/gitlab/jobs/8503792002"]
}
```

### Example: company careers profile

```json
{
  "mode": "companyProfile"
}
```

## Use cases

- **Recruiting research** — track which roles GitLab has open in a specific department or location
- **Talent market monitoring** — watch `firstPublishedAt`/`updatedAt` to see how fast GitLab is hiring in a function
- **Sales intelligence** — filter by `quotaCoverageType` to see GitLab's current sales-role mix
- **Job board aggregation** — feed GitLab's own postings into a broader jobs dataset alongside other companies
- **Employer-brand research** — pull GitLab's careers-page summary and values/handbook links in one call

## FAQ

**Is this affiliated with GitLab?**
No. This is an independent, third-party actor that reads GitLab's own publicly posted job board and careers page; it is not affiliated with or endorsed by GitLab.

**Do I need a login or API key?**
No. GitLab's job board and careers page are public, and the actor reads them without authentication.

**What's the difference between `searchJobs` and `allJobs`?**
`searchJobs` matches a keyword against title, company, location, and requisition ID. `allJobs` returns open postings without requiring a keyword, so you can apply only the other filters (location, department, dates).

**Why would I use `byJobIds` or `byJobUrls` instead of search?**
Use them when you already know the exact job(s) you want — they resolve a specific posting directly rather than searching or listing.

**Why are some job fields like `descriptionHtml` missing?**
`descriptionHtml`, `descriptionText`, `sectionHeadings`, `descriptionLinks`, and `complianceTypes` all come from the job's detail page, so they're only present when `fetchFullDescription` is enabled (or when `departmentNames` filtering forces a detail fetch anyway). `departmentNames`/`officeNames` come from the job listing itself and appear regardless of `fetchFullDescription`, when GitLab's listing publishes them for that job.

**What happens if I pass an invalid job ID or URL?**
The actor returns a typed `error` record for that lookup instead of failing the whole run, so valid lookups in the same input still come back.

**How fresh are the job listings?**
Every run queries GitLab's live job board directly, so results reflect whatever is posted at run time.

**What does the `companyProfile` mode actually return?**
A single summary record built from GitLab's own careers page — its intro paragraphs, section headings, and links to its benefits, values, and handbook pages — rather than individual job postings.
