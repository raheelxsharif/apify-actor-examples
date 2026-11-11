# Bayt.com Job Finder: Pay, Skills & Companies

Bayt.com is the largest job board across the Middle East and North Africa. This actor turns its public job pages into a clean dataset you can filter, chart or load into a CRM — with local-currency pay, a USD monthly range, exact posting timestamps, full descriptions, skills, preferred-candidate criteria and company profiles.

## What it does

- Searches any of 24 Bayt markets by keyword, city and server-side filters.
- Accepts pasted job, search and company URLs, or a list of job IDs.
- Pulls a company profile (website, size, industry, followers, about text) and optionally every open role at that company.
- Parses salary twice: the native range shown on the job (`AED 1,836 - AED 3,673`) and the USD monthly bucket Bayt publishes in its structured data.
- Keeps the exact posting timestamp from the listing card, plus a date-only field for older integrations.
- Removes duplicate jobs across queries and pages, and never emits a placeholder or "blocked" record.

## Services

Use the output to:

- build a daily feed of new roles for one market or one employer;
- compare advertised pay for the same title across countries and cities;
- create recruiting or sales lead lists from company profiles and hiring activity;
- research which skills, degrees and nationalities employers ask for;
- monitor a company's open roles over time.

## How it works

Bayt serves server-rendered HTML. The actor builds Bayt's own SEO search paths (`/en/<market>/jobs/<keyword>-jobs/`, city paths and keyword-plus-city paths), adds Bayt's server-side filter and sort parameters, and reads 30 cards per page. Listing pages provide the job ID, title, company, location, listing salary, remote tag, posting epoch and aggregated-job flag. When detail enrichment is on, each job page adds its `JobPosting` JSON-LD block, structured `data-automation-id` rows, preferred-candidate pairs, description and skills. Company mode reads the profile chips and uses Bayt's company filter (`gid_pid`) for the employer's open roles.

Requests are polite and sequential. Detail pages are optional, so a listing-only run is the cheapest way to collect a large feed. If a page is temporarily unavailable, the actor retries; a job whose detail page cannot be loaded is still returned with its listing fields instead of being dropped.

## Input

| Field | Default | What it does |
|---|---|---|
| `mode` | `search` | `search`, `byUrls`, `byJobIds` or `company` |
| `searchQueries` | `["accountant"]` | One or more keywords; each runs as its own Bayt search |
| `country` | `international` | 24 markets: International, UAE, Saudi Arabia, Qatar, Kuwait, Bahrain, Oman, Egypt, Jordan, Lebanon, Iraq, Morocco, Tunisia, Algeria, Libya, Syria, Palestine, Pakistan, India, USA, UK, Canada, Australia, Türkiye |
| `city` | – | Optional city such as Dubai, Riyadh or Cairo |
| `employmentType` | `any` | Full time, part time, contractor, temporary, internship or freelance |
| `careerLevel` | `any` | Student, fresh graduate, entry, mid career, management, director or senior executive |
| `datePosted` | `any` | Last 24 hours, 7 days or 30 days |
| `sortBy` | `date` | Newest first or most relevant |
| `industry` | `any` | 22 Bayt employer industries |
| `companyName` | – | Employer-name facet |
| `remoteOnly` | `false` | Bayt's remote/hybrid filter |
| `easyApplyOnly` | `false` | Jobs that accept an application on Bayt |
| `minSalary` | – | USD monthly floor; jobs without a published USD range are excluded |
| `includeDetails` | `true` | Visit each job page for full content and parsed pay |
| `maxItems` | `50` | Overall record limit, up to 10,000 |
| `maxPages` | `100` | Page cap per search, up to 400 |
| `startUrls` | `[]` | Job, search or company URLs (Paste URLs mode) |
| `jobIds` | `[]` | Job IDs or full job URLs (Resolve IDs mode) |
| `companyUrls` | `[]` | Company profile URLs or numeric IDs (Company mode) |
| `includeCompanyJobs` | `true` | Also return the company's open roles |

### Example: keyword search in one market

```json
{
  "mode": "search",
  "searchQueries": ["accountant", "financial analyst"],
  "country": "uae",
  "datePosted": "last_week",
  "employmentType": "full_time",
  "maxItems": 100
}
```

### Example: salary-floor search

```json
{
  "mode": "search",
  "searchQueries": ["software engineer"],
  "country": "saudi-arabia",
  "minSalary": 3000,
  "includeDetails": true,
  "maxItems": 50
}
```

### Example: paste mixed URLs

```json
{
  "mode": "byUrls",
  "startUrls": [
    "https://www.bayt.com/en/uae/jobs/accountant-jobs/",
    "https://www.bayt.com/en/uae/jobs/senior-accountant-5499441/",
    "https://www.bayt.com/en/company/al-futtaim-group-1661592/"
  ],
  "maxItems": 60
}
```

### Example: company profile and its jobs

```json
{
  "mode": "company",
  "companyUrls": ["https://www.bayt.com/en/company/al-futtaim-group-1661592/"],
  "includeCompanyJobs": true,
  "maxItems": 50
}
```

The default prefill is the smallest useful run: one search, one page, listing cards only.

## Output

Each job record uses `type: "job_bayt"` and `recordType: "job"`. Common fields:

- Identity: `jobId`, `url`, `applyUrl`, `applyType`, `easyApply`, `directApply`, `isExternalJob`, `isAggregatedJob`
- Role: `title`, `company`, `companyId`, `companyProfileUrl`, `companyLogo`, `companySize`, `companyEmployeesSize`, `companyIndustry`, `industry`, `industryId`, `industryName`
- Location: `location`, `city`, `region`, `country`, `countryCode`, `isRemote`, `workplaceType`, `jobType`
- Dates: `postedAt`, `postedAtTimestamp`, `postedAtRelative`, `datePosted`, `expiresAt`, `scrapedAt`
- Requirements: `employmentType`, `careerLevel`, `experience`, `experienceMonths`, `vacancies`, `skills`, `skillsText`, `preferredCandidate`, `gender`, `nationality`, `residenceCountry`, `degree`, `major`, `ageRange`, `candidateStatus`
- Pay: `salary`, `salaryMin`, `salaryMax`, `salaryCurrency`, `salaryPeriod`, `salaryUsdMin`, `salaryUsdMax`, `salaryUsdCurrency`, `salaryUsdPeriod`, `minSalaryUSD`, `maxSalaryUSD`
- Content: `descriptionHtml`, `descriptionText`, `listingSummary`
- Provenance: `searchKeyword`, `searchCountry`, `searchCity`, `searchUrl`, `page`, `rank`

Company records use `type: "company_bayt"` and include `companyId`, `company`, `companyProfileUrl`, `companyWebsite`, `companySize`, `companyIndustry`, `companyFollowers`, `companyAbout`, `location`, `city`, `country`, `openJobsCount` and `scrapedAt`.

Fields the source does not publish are omitted. Booleans such as `easyApply` and `isAggregatedJob` are always present on job records.

## Usage cost

You pay for Apify platform compute, storage and any managed proxy requests used to reach Bayt; the job data itself is free. A listing-only run uses one request per 30 jobs; turning on `includeDetails` adds one request per job and returns much more content. Keep `maxItems` and `maxPages` as small as your use case needs, and run large history backfills in slices by city, keyword or date.

## FAQ

**Is this affiliated with Bayt.com?**
No. It reads publicly visible Bayt pages and has no relationship with Bayt.

**Do I need a Bayt account or cookies?**
No. No login, cookie or API key is required.

**Which countries are covered?**
The 24 markets in the Market dropdown. Other Bayt country paths can still be scraped with Paste URLs mode.

**Why is salary missing on many jobs?**
Bayt only publishes pay on a minority of posts, and imported ("aggregated") listings usually carry no salary. The actor never guesses pay from description text.

**Why do some records have fewer fields?**
Jobs imported from employer sites carry a description, location, industry and closing date, but usually no salary, career level, skills or company profile. They are marked `isAggregatedJob: true`.

**Why can a search stop before maxItems?**
Bayt exposes about 10,000 results per single search. The actor stops at that ceiling or when a page repeats, and says so in the run's status message. Split by city, keyword or date to collect more.

**How current are the jobs?**
Every run reads Bayt live. `postedAtTimestamp` is the exact listing timestamp for normal search results; for a job opened directly by ID, Bayt only exposes the posting date, so `postedAt` is returned without an exact time.

**Can I use my old inputs?**
Yes. Existing `startUrl`, `keyword`, `locationFilter` and `resultsWanted` tasks are mapped automatically in code. The old `proxyConfiguration` field is accepted but ignored because access is managed by the actor.

**Can I scrape one job?**
Yes. Use Resolve job IDs with the numeric ID or full URL, or paste the URL in Paste URLs mode.

**Do I need to configure a proxy?**
No. The actor manages access to Bayt automatically, so there is no proxy field to set. When Bayt blocks a direct request, the actor uses a managed proxy and those proxy requests are billed with the run.

## Notes

- Public pages only; no login or member-only content is accessed.
- Arabic and English Bayt URLs are accepted in Paste URLs and Resolve job IDs.
- Descriptions are returned as HTML and plain text. Some Bayt posts are machine-translated; the page says so when that applies.
- Record order follows Bayt's own result order for each search, with `rank` recording the position.
- The actor removes duplicate job IDs across queries, pages and URL lists.

## Limitations

- Pay, career level, skills and preferred-candidate criteria are conditional: they appear only when the employer published them.
- Aggregated listings (8-digit job IDs) have less metadata by design; use `isAggregatedJob` to filter them.
- One search can expose at most about 10,000 results. Partition broad searches to go deeper.
- Some jobs have no closing date or no company profile; those fields are omitted.
- A city that Bayt does not recognise falls back to the country search rather than failing the run.
- A job opened directly by ID exposes Bayt's posting date but not the exact time; exact timestamps come from search results, where the listing epoch is available.
- The dataset reflects what Bayt shows at run time. A job opened directly by ID is skipped when its page is gone; a job seen in a listing keeps its listing fields if detail enrichment fails.
