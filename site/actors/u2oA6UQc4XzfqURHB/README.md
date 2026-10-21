# We Work Remotely Job Listings & Salary Extractor

Collect every live posting on **We Work Remotely** from the board's 12 official category RSS feeds, enriched by the public job XML. Get titles, companies, categories, employment types, applicant regions and countries, full descriptions, skills, salary ranges parsed from the posting text, company profiles, real application links or emails, and current listing and expiry dates. No login, cookies, API key, or proxy is required.

## What this actor does

- **Full live inventory:** reads all 12 category feeds in one run (447 unique openings on 2026-10-20), deduplicated by job URL.
- **Three ways to work:** browse categories, search the whole board by keyword, or paste job, category, search, company and listing URLs.
- **Useful filters:** category, employment type, applicant region, applicant country, listing window, minimum/maximum salary, and salary-only.
- **Salary extraction:** parses min/max, currency and pay period when a posting states a range; keeps the original salary text; never invents a number.
- **Location eligibility:** separates jobs open worldwide from jobs restricted to specific countries, and exposes ISO country codes.
- **Complete job content:** full description HTML and plain text, source-provided skills, company website, logo and profile.
- **Application details:** the public job XML carries the employer's application URL or email for almost every posting, including roles whose on-page apply button is behind a We Work Remotely account.
- **Correct dates:** current listing date, original first-post date and expiry date, with a renewal flag for reposted roles.
- **Monitoring:** optional incremental mode returns only new or renewed jobs since the previous run.
- Empty fields are omitted from each record.

## Output per job

**Identity**

- `type` — always `job_wwr`
- `url` — canonical We Work Remotely job page
- `id`, `slug` — job slug from the URL
- `title`, `role` — job title
- `company`, `companySlug`, `companyProfileUrl` — employer and its We Work Remotely company page
- `category`, `subcategory` — We Work Remotely category and the Product Design / UX-UI subcategory when applicable
- `feedSlugs` — category feeds where the job appeared

**Employment and location**

- `employmentType`, `employmentTypeNormalized` — e.g. `Full-Time` and `full-time`
- `jobLocationType` — `TELECOMMUTE`
- `region` — the board's region label, e.g. `Anywhere in the World`, `USA Only`, `Europe`
- `country`, `countryCodes` — applicant countries named in the posting, plus ISO-2 codes
- `isWorldwide` — true when no country restriction is published
- `locationText` — readable location restriction
- `applicantCountries`, `applicantLocationRequirements` — applicant country names

**Dates**

- `postedAt` — current listing date (renewed listings get a fresh date)
- `firstPostedAt` — original first-post date
- `postedAgeDays` — days since the current listing date
- `isRenewed` — true when the current listing is newer than the original post
- `expiresAt`, `applyBefore`, `validThrough` — listing expiry
- `scrapedAt` — collection timestamp

**Salary**

- `salary` — the salary text found in the posting
- `salaryDetails` — `{min, max, currency, unit}` object
- `salaryMin`, `salaryMax`, `salaryCurrency`, `salaryUnit` — flattened values (`salaryUnit` is `YEAR`, `MONTH`, `WEEK`, `DAY` or `HOUR`)
- `salaryPeriod`, `salaryType` — lowercase period and `base`, `ote`, `total-comp` or `hourly`
- `salaryAnnualMin`, `salaryAnnualMax` — annualized values used by the salary filters
- `salarySource` — `description` or `detail`

**Description and skills**

- `skills`, `skillsRaw` — tags supplied by the employer, when present
- `descriptionHtml`, `descriptionText` — full posting content; `cleanHtml` removes scripts and tracking pixels

**Company profile**

- `companyHq`, `companyWebsite`, `companyLogo`, `companyLogoFull`
- `companyBio`, `companyStatement`, `companyJobsCount`, `companyTop100`, `companyFoundedYear`
- `hiringOrganizationSameAs` — additional company links from the job page, when present

**Application and monitoring**

- `applyUrl`, `applyEmail`, `applyUrlSource` — the employer's real application URL or email from the public job XML
- `directApply` — whether the posting advertises direct application
- `newSinceLastRun`, `firstSeenAt` — incremental mode only
- `sourceUrl` — the feed URL the record came from

## Input

| Field | Type | Default | Description |
| --- | --- | --- | --- |
| `mode` | string | `browse` | `browse`, `search` or `byUrls` |
| `categories` | array | all | One or more of the 12 category feeds |
| `searchTerm` | string | – | Words to match in title, company, category or description; all words must match. Required in `search` mode |
| `startUrls` | array | – | Job, category, search, company or listing URLs (`byUrls` mode) |
| `employmentTypes` | array | all | `Full-Time`, `Contract` |
| `regions` | array | all | Region labels such as `Anywhere in the World`, `USA Only`, `Europe` |
| `countries` | array | all | Applicant countries as ISO-2 codes (e.g. `US`, `DE`, `IN`) |
| `postedWithin` | string | `any` | `any`, `1`, `3`, `7`, `14` or `30` days |
| `minSalary` | integer | – | Minimum annualized salary in USD |
| `maxSalary` | integer | – | Maximum annualized salary in USD |
| `requireSalary` | boolean | `false` | Only keep jobs with a confidently parsed salary |
| `includeDescription` | boolean | `true` | Include full description HTML and text |
| `cleanHtml` | boolean | `true` | Remove scripts, tracking pixels and unsafe attributes |
| `includeDetails` | boolean | `false` | Also fetch each job's HTML page for extra structured fields; the public job XML (apply URL/email, company bio, exact dates) is always read |
| `includeCompanyProfile` | boolean | `true` | Fetch company website, logo, about text and open-job count |
| `incremental` | boolean | `false` | Return only new or renewed jobs since the last run |
| `stateKey` | string | – | Optional name for an incremental campaign |
| `maxItems` | integer | `200` | Maximum jobs to return (1-5000) |

### Example: every programming job

```json
{
  "mode": "browse",
  "categories": [
    "remote-full-stack-programming-jobs",
    "remote-front-end-programming-jobs",
    "remote-back-end-programming-jobs"
  ],
  "maxItems": 200
}
```

### Example: Python roles open to Pakistan, above $60k

```json
{
  "mode": "search",
  "searchTerm": "python",
  "countries": ["PK"],
  "minSalary": 60000,
  "postedWithin": "14",
  "maxItems": 50
}
```

### Example: paste a company and a job URL

```json
{
  "mode": "byUrls",
  "startUrls": [
    "https://weworkremotely.com/company/stripe",
    "https://weworkremotely.com/remote-jobs/stripe-abuse-research-engineer"
  ]
}
```

### Example: daily new-job monitor

```json
{
  "mode": "search",
  "searchTerm": "product manager",
  "incremental": true,
  "stateKey": "product-manager-monitor",
  "maxItems": 100
}
```

## Use cases

- **Recruiters and sourcers** — monitor new remote roles by category, country eligibility and salary.
- **Job boards and newsletters** — sync the full We Work Remotely inventory with full descriptions and company profiles.
- **Salary research** — compare disclosed remote pay by role, company and region.
- **Market analysis** — track which categories and countries are hiring remotely.
- **Lead generation** — identify remote-first companies, their websites and open-role counts.
- **Job alerts** — run incremental searches and push only new or renewed postings into a workflow.

## Limitations

- We Work Remotely does not publish salary for every job. The actor parses a range only when the posting states one (about six in ten postings), so salary fields are absent on non-disclosing jobs.
- Almost every public job XML carries an application URL or email; a small minority only exposes the We Work Remotely account wall, in which case both are omitted and `url` opens the posting.
- Skills are optional on the source and appear on a minority of postings. They are returned as published and are not used for keyword matching.
- Company bios are available for about half of postings; where the board has no profile text the field is omitted.
- Salary filters compare the published annualized numbers; the actor does not convert between currencies.

## FAQ

### Where does the data come from?

The 12 public category RSS feeds published by We Work Remotely, plus job and company pages when detail or company enrichment is enabled. Links back to We Work Remotely are preserved in every record. No affiliation with We Work Remotely.

### Do I need a proxy, login or API key?

No. The public feeds and pages used here work without credentials or proxy setup.

### Why is salary missing on some jobs?

About six in ten postings state a numeric range. The actor returns a salary only when it can parse one confidently and never substitutes a guess. Use `requireSalary` to keep only disclosing jobs.

### Why is `applyUrl` missing on most jobs?

The public job XML exposes the employer's application link or email for almost every posting, even when the on-page apply button is locked. The actor returns `applyUrl` for web links and `applyEmail` for email applications. If neither exists, use `url` to open the posting.

### What is the difference between `postedAt` and `firstPostedAt`?

We Work Remotely renews listings, so the visible "Posted" date can be newer than the original publication date. `postedAt` is the current listing date used by `postedWithin`; `firstPostedAt` is the original post date. `isRenewed` marks the difference.

### How do region and country filters work?

A job open worldwide matches any selected region or country because it is eligible everywhere. A restricted job matches only when its published region or applicant-country list overlaps your selection. The applicant-country list takes precedence over the broader region label.

### Are all skills returned?

No. We Work Remotely's skill tags are optional and appear on a minority of jobs. They are returned when present, exactly as published.

### How do I get only new jobs?

Set `incremental` to `true` and give the campaign a `stateKey`. The next run of the same search returns only jobs that were not seen before, plus renewed listings. Use a different `stateKey` for a different search.

### How fresh is the data?

The feeds are read live at run time and update continuously. Schedule the actor as often as you need; incremental mode keeps follow-up runs small.

### Is this affiliated with We Work Remotely?

No. This is a third-party actor that reads publicly available We Work Remotely feeds and pages.
