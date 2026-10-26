# Greenhouse ATS Job Board Lookup by Company

Pull every open role from any company hiring on Greenhouse, or look up specific postings by URL. Start from a board token, a careers link, a company name or a domain; get clean job records with structured pay ranges, locations, departments, application questions, published dates and direct apply links. Greenhouse's public job board data is read directly, with no login, API key, cookie or proxy.

## What this actor does

- **Three modes:** scrape company boards, fetch specific job URLs, or resolve company boards from names and domains.
- **Flexible company input:** board tokens, `boards.greenhouse.io` / `job-boards.greenhouse.io` links, embed links, single-job links, company names and domains.
- **Structured pay:** every disclosed pay band is preserved with min/max, currency, period and base/OTE labelling.
- **Useful filters:** title, location, department, workplace type, employment type, published/updated date, salary and currency.
- **Descriptions and questions:** full HTML, plain text or Markdown, plus the application questions each posting asks.
- **Complete context:** published and updated dates, offices, department hierarchy, metadata, language, requisition ID and apply URL.
- **Empty fields are omitted** — not set to `null`, `""` or a placeholder.

## Output per job

- `id` — Greenhouse job post ID
- `internalJobId` — Greenhouse's internal job ID
- `requisitionId` — employer requisition number, when published
- `title` — job title
- `companySlug` — Greenhouse board token
- `companyName` — employer name
- `location` — primary location object with `name`, `city`, `state`, `country`
- `locations[]` — every location when a posting covers several
- `isRemote` — `true` when the posting is remote
- `workplaceType` — `remote`, `hybrid` or `onsite` when stated
- `departments[]` — department names
- `departmentsDetailed[]` — department `id`, `name` and `parentId`
- `offices[]` — office names
- `officesDetailed[]` — office `id`, `name`, `location` and `parentId`
- `employmentType` — full-time, part-time, contract, internship or temporary, when stated
- `educationLevel` — Greenhouse education requirement code, when published
- `salary` — primary pay object with `min`, `max`, `currency`, `period`, `range` and `source`
- `salaryBands[]` — every disclosed band with `label`, `min`, `max`, `currency`, `period`, `kind` and `source`
- `publishedAt` — first published timestamp
- `updatedAt` — last updated timestamp
- `applicationDeadline` — application deadline, when set
- `language` — posting language code
- `metadata[]` — custom fields with `name`, `value` and `valueType`
- `team` — team name from metadata, when present
- `seniority` — seniority from metadata, when present
- `content` — full description HTML, when descriptions are enabled
- `contentText` — plain-text description, when requested
- `contentMarkdown` — Markdown description, when requested
- `questions[]` — application questions with `label`, `required` and `fields`
- `applyUrl` — direct apply link
- `absoluteUrl` — Greenhouse-hosted posting URL
- `boardUrl` — company's Greenhouse board URL
- `sourceUrl` — stable Greenhouse posting URL
- `region` — `us` or `eu`
- `recordType` — `job`
- `scrapedAt` — UTC timestamp of the scrape

## Output per company (mode = `companySearch`)

- `companySlug`, `companyName`, `companyDescription` (when published)
- `boardUrl`, `region`, `openJobs` (open jobs when `verifyCompanies` is true)
- `status` — `found`, `empty`, `not_found` or `ambiguous`
- `sourceUrl`, `recordType: "company"`, `scrapedAt`

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `companyJobs` | `companyJobs`, `jobDetails` or `companySearch` (legacy `getJobs`/`getJob` aliases are accepted) |
| `companies` | array | – | Tokens, board URLs, job URLs, company names or domains |
| `jobUrls` | array | – | Full job URLs for `jobDetails` |
| `searchQuery` | string | – | Directory search text for `companySearch` |
| `titleKeywords` | array | – | Keep titles containing any of these words |
| `excludeTitleKeywords` | array | – | Drop titles containing any of these words |
| `locations` | array | – | Keep locations or offices containing any of these words |
| `departments` | array | – | Keep departments containing any of these words |
| `workplaceType` | string | `any` | `any`, `remote`, `hybrid` or `onsite` |
| `employmentType` | string | `any` | `any`, `full-time`, `part-time`, `contract`, `internship` or `temporary` |
| `publishedAfter` | string | – | First-published cutoff (`YYYY-MM-DD`) |
| `updatedAfter` | string | – | Last-updated cutoff (`YYYY-MM-DD`) |
| `onlyWithSalary` | boolean | `false` | Keep only jobs with a stated pay range |
| `minAnnualSalary` | integer | – | Minimum annualized base pay |
| `salaryCurrency` | string | `any` | Only jobs with pay in this currency |
| `sortBy` | string | `board` | `board`, `publishedNewest`, `publishedOldest`, `updatedNewest` or `titleAZ` |
| `content` | boolean | `false` | Include full job descriptions |
| `descriptionFormat` | string | `html` | `html`, `text`, `markdown` or `all` |
| `includeQuestions` | boolean | `false` | Fetch application questions (one extra request per emitted job) |
| `maxItems` | integer | `500` | Maximum records to emit (1–10,000) |
| `maxJobsPerCompany` | integer | `0` | Per-company cap after filters; `0` means no cap |
| `verifyCompanies` | boolean | `true` | Live-check companies in `companySearch` and report open jobs |

### Example: all engineering roles at three companies

```json
{
  "mode": "companyJobs",
  "companies": ["stripe", "figma", "https://job-boards.greenhouse.io/airbnb"],
  "titleKeywords": ["engineer", "developer"],
  "workplaceType": "remote",
  "content": false,
  "maxItems": 200
}
```

### Example: company names and domains

```json
{
  "mode": "companyJobs",
  "companies": ["Anthropic", "jobs.elastic.co", "HubSpot"],
  "onlyWithSalary": true,
  "minAnnualSalary": 120000,
  "salaryCurrency": "USD",
  "sortBy": "publishedNewest",
  "maxItems": 100
}
```

### Example: specific jobs with descriptions, Markdown and questions

```json
{
  "mode": "jobDetails",
  "jobUrls": [
    "https://job-boards.greenhouse.io/gitlab/jobs/8860302002",
    "https://boards.greenhouse.io/figma/jobs/5426468004"
  ],
  "content": true,
  "descriptionFormat": "all",
  "includeQuestions": true
}
```

### Example: resolve a company list

```json
{
  "mode": "companySearch",
  "companies": ["Airbnb", "stripe.com", "HubSpot", "not-a-real-board"],
  "verifyCompanies": true,
  "maxItems": 10
}
```

## Use cases

- **Job boards and aggregators** — build live company career pages from the source, with stable IDs and apply links.
- **Recruiting and sourcing** — monitor target companies by department, location and seniority.
- **Compensation research** — collect disclosed pay bands with currency and period labels.
- **Sales and hiring signals** — see which teams and locations are expanding.
- **AI agents and enrichment** — feed clean job text, questions and structured pay into matching or RAG pipelines.
- **Applicant tooling** — list the exact questions a Greenhouse application will ask before a candidate starts.

## FAQ

**Do I need a Greenhouse account, API key or proxy?**
No. The actor reads publicly published job boards. There is nothing to configure and no login, cookie or proxy needed.

**How do I find a company's board token?**
Paste the company's Greenhouse board link, embed link or a single job link. If you only have a company name or domain, the actor tries the built-in verified directory, then common board-name variants. Name/domain lookup is best-effort; a board URL or token is always exact.

**Does this work for companies on the EU Greenhouse host?**
Yes. EU boards return the same public data and are marked `region: "eu"`.

**Why is salary missing from some jobs?**
Pay disclosure is a company choice. Many employers publish no range at all; the field is included only when a real range is available. The actor never invents a number.

**What is the difference between `salary` and `salaryBands`?**
`salary` is the primary base-pay summary. `salaryBands` keeps every disclosed band, including location zones and OTE ranges, because one posting can publish several. OTE and commission bands stay in `salaryBands` but are excluded from the `salary` envelope.

**Why is employment type missing?**
Greenhouse no longer exposes a dedicated employment-type field. The actor reads it from employer metadata when present and omits it otherwise.

**Can I get the full job description and questions?**
Yes. Turn on descriptions and choose HTML, text, Markdown or all three. Questions are optional because they require one additional request per job.

**How many jobs can I get per company?**
A company's whole board is returned in one response, so there is no page limit. Use `maxItems` and `maxJobsPerCompany` to control run size.

**Does this work for private or unlisted boards?**
No. Only publicly published Greenhouse boards are readable.

**Is this affiliated with Greenhouse?**
No. This is an independent third-party actor that reads publicly published job board data and is not operated by or affiliated with Greenhouse Software, Inc.

## Limitations

- Pay, employment type and some metadata are employer-optional; missing values are real, not parsing failures.
- Company name/domain resolution is best-effort. Use a board URL or token for an exact match.
- Application questions are only available per job and add one request per record.
- A posting can publish multiple pay bands; the actor preserves all of them instead of picking one.
