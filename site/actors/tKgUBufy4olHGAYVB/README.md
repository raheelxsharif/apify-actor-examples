# SimplyHired Job Search & Salary Parser

SimplyHired gathers job postings from employer career sites, staffing agencies, and niche job boards across 23 country markets. This actor turns those postings into structured records: it can run keyword and location searches (or replay search URLs you paste in), or fetch individual job postings you already have, and it returns each listing with its salary parsed into numbers, the full description, coordinates, and application details.

## What this actor does

- **Search any SimplyHired market:** 23 country sites (United States, Canada, United Kingdom, Germany, France, India, Japan, Australia, Brazil, and more), each with keyword, location, and radius search.
- **Filter server-side:** date posted, employment type, minimum pay, remote/hybrid, sort order, and developer skills (one or several) are applied by SimplyHired itself, so the result count matches the site.
- **Enrich every job, not just the first one:** each record is completed from the posting's own detail data — full description, salary numbers, qualifications, coordinates, employer rating, and application link.
- **Parse salaries in local formats:** display text like `$140,000 - $180,000 a year`, `£45,000 - £55,000 a year`, `80.000 € – 120.000 € pro Jahr`, or `時給 4,000円` becomes numeric `salaryMin` / `salaryMax` with the right currency and pay period.
- **Maximize coverage on big searches:** SimplyHired serves roughly 900–1,000 results per search. With `maximizeCoverage` the actor keeps going with extra facet searches (job types, developer skills) and merges the unique postings — 2–2.5x more jobs than a single search in testing.
- **Optional employer company pages:** with `enrichCompanyPages` the actor looks up each employer's SimplyHired company page (once per employer) and adds its address, headcount band, revenue band, and industry — only when the page's own company name matches the listing.
- **Feed URLs as well as forms:** paste SimplyHired search URLs or job URLs (or bare job IDs) and let the actor do the pagination.
- **Fields a listing doesn't provide are left out of that record** — no empty placeholders.

## Output per job

| Field | Description |
|---|---|
| `jobId` | Unique SimplyHired job key. |
| `title`, `normalizedTitle` | Job title as published and its normalized form. |
| `company` | Employer name. |
| `location` | Location string as shown on the listing. |
| `city`, `state` | City and state/region parsed from the listing. |
| `latitude`, `longitude` | Coordinates of the posting location. |
| `remoteType` | Remote or hybrid arrangement, when stated. |
| `employmentType` | Primary employment type (Full-time, Contract, Part-time, …). |
| `employmentTypes` | Every employment type on the posting, including the listing's localized labels when provided. |
| `salary` | Salary text exactly as displayed on the listing. |
| `salaryMin`, `salaryMax` | Parsed pay bounds in full local-currency units. |
| `salaryCurrency` | ISO currency code (e.g. `USD`, `GBP`, `EUR`, `JPY`). |
| `salaryPeriod` | `annual`, `monthly`, `weekly`, `daily`, or `hourly`. |
| `description` | Full job description with HTML tags stripped. |
| `benefits` | Benefits listed on the posting. |
| `qualifications` | Qualifications extracted by SimplyHired. |
| `requirements` | Requirement tags shown on the search card. |
| `datePosted` | ISO 8601 UTC timestamp of the posting date. |
| `expirationDate` | Listing expiration date, when stated. |
| `jobUrl` | SimplyHired listing URL. |
| `applicationUrl` | SimplyHired apply/redirect link for the posting. |
| `companyPageUrl` | Employer's SimplyHired company page (from the source, or verified by `enrichCompanyPages`). |
| `companyAddress`, `companyEmployeeCount`, `companyRevenue`, `companyIndustry` | Employer profile details from the company page (only with `enrichCompanyPages`, and only when the page matches the listing's employer). |
| `companyRating` | Employer rating, when SimplyHired shows one. |
| `sponsored` | Whether the listing is a sponsored placement. |
| `indeedApply` | Whether the posting uses the Indeed application flow. |
| `snippet` | Short preview text from the search card. |
| `logoUrl` | Employer logo image, when available. |
| `sourceDomain` | Which SimplyHired site the record came from (e.g. `www.simplyhired.co.uk`). |
| `searchUrl`, `searchQuery` | The search that produced the record. |
| `country` | Country market value (e.g. `united-kingdom`). |
| `sourceUrl` | URL the record was scraped from. |
| `recordType` | Always `"job"`. |
| `scrapedAt` | ISO 8601 UTC timestamp of when this actor fetched the record. |

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | String (select) | `search` | `search` (form-driven searches) or `byUrl` (scrape specific URLs). |
| `searchQueries` | Array of strings | – | Job titles, skills, or keywords. Each entry runs its own search. Used when `mode` is `search` and no search URLs are given. |
| `searchUrls` | Array of strings | – | Full SimplyHired search URLs (with any filters). They take precedence over the query/location/filter fields. |
| `jobUrls` | Array of strings | – | Job URLs, job IDs, or search URLs to scrape directly. Used when `mode` is `byUrl`. |
| `location` | String | – | City, state, ZIP, or `remote`. Leave empty for a nationwide search. |
| `country` | String (select) | `united-states` | Which SimplyHired site to search. Pick the country that matches your location. |
| `radius` | String (select) | site default (25 mi) | Distance around the location: exact, 5, 10, 15, 25, 50, or 100 miles. |
| `sortBy` | String (select) | `relevance` | `relevance`, `date` (newest first), or `distance` (nearest first). |
| `datePosted` | String (select) | any time | Past 24 hours, 3 days, week, 2 weeks, or month. |
| `employmentType` | String (select) | any | Full-time, Contract, Part-time, Temporary, Permanent, Internship, Temp-to-hire, or Freelance. |
| `minSalary` | Integer | `0` | Minimum pay in the country's local currency (e.g. `100000` on the US site = $100,000/year). |
| `remoteType` | String (select) | any | `Remote` or `Hybrid work`. |
| `developerSkill` | String (select) | any | One software skill such as Python, Java, React, SQL, Go, or Swift. |
| `developerSkills` | Array of strings | – | Extra skills to require together with the single skill above (AND). Accepts names (`Python`, `SQL`) or codes. |
| `fetchJobDetails` | Boolean | `true` | Fetch each posting's detail data (description, parsed salary, coordinates, qualifications). Adds one fast request per job. |
| `enrichCompanyPages` | Boolean | `false` | Look up each employer's SimplyHired company page (one request per employer, cached) for address, headcount band, revenue band, and industry. Only matched employers are filled. |
| `maximizeCoverage` | Boolean | `false` | Keep collecting past SimplyHired's per-search result limit using extra facet searches, merged and deduplicated. Slower. |
| `maxItems` | Integer | `20` | Hard cap on returned records (1–5000). |

## Examples

### Keyword search with filters

```json
{
    "mode": "search",
    "searchQueries": ["registered nurse"],
    "location": "Houston, TX",
    "country": "united-states",
    "datePosted": "7",
    "employmentType": "CF3CP",
    "maxItems": 100
}
```

### Remote jobs in Canada above a pay floor

```json
{
    "mode": "search",
    "searchQueries": ["customer success manager"],
    "location": "remote",
    "country": "canada",
    "remoteType": "Remote",
    "minSalary": 80000,
    "sortBy": "date",
    "maxItems": 150
}
```

### Run prepared search URLs (any country)

```json
{
    "mode": "search",
    "searchUrls": [
        "https://www.simplyhired.co.uk/search?q=software+engineer&l=London&t=7",
        "https://www.simplyhired.de/search?q=entwickler&l=Berlin"
    ],
    "maxItems": 200
}
```

### Scrape specific job URLs

```json
{
    "mode": "byUrl",
    "jobUrls": [
        "https://www.simplyhired.com/job/pR885UfXXEquRuih1PVGw9Ujl7fkq-RglEg5fxkaZzhQn1D46UaJzQ",
        "https://www.simplyhired.ca/job/MebGsOvO4rbcC1COSt7eX0Ii5i-RX6ItIdOtA-OMQGDW5w024IZ81g"
    ],
    "maxItems": 2
}
```

## Use cases

- **Pay benchmarking:** aggregate `salaryMin` / `salaryMax` by role, city, and employment type within one currency.
- **Cross-market hiring research:** compare supply for the same role across the 23 SimplyHired markets in separate runs.
- **Recruiting pipelines:** monitor which employers are posting for a role, with coordinates and company ratings attached.
- **Job alerting and feeds:** schedule a run per keyword and forward new `jobId`s to your CRM, sheet, or webhook.
- **Skills analysis:** use `developerSkill` filters and `qualifications` to track technology demand over time.
- **Posting refresh:** feed previously collected `jobUrl` values back through `mode: "byUrl"` to re-check status and pay.

## FAQ

### Where does this data come from?

Every record comes from SimplyHired's public job pages and the site's own page data, without any login. This actor is an independent tool and is not affiliated with or endorsed by SimplyHired.

### Which countries are supported?

The United States, Canada, the United Kingdom, Ireland, Germany, France, Spain, Italy, the Netherlands, Belgium, Austria, Switzerland, Portugal, Sweden, India, Japan, South Korea, China, Brazil, Mexico, Australia, Argentina, and South Africa. Choose the country in the input, or use a search URL from the market you want.

### Why are some salary fields missing?

SimplyHired only shows pay when the employer discloses it. When a listing has no pay information, the salary fields are omitted from that record rather than guessed. `salary` keeps the exact text; `salaryMin` / `salaryMax` are parsed from it (or from SimplyHired's structured pay data on the US site) in the local currency.

### What's the difference between `salary` and `salaryMin` / `salaryMax`?

`salary` is the display text exactly as shown (e.g. `£45,000 - £55,000 a year`). `salaryMin` / `salaryMax` / `salaryCurrency` / `salaryPeriod` are the same value parsed into numbers, a currency code, and a pay period for filtering and aggregation. Use `salaryCurrency` before comparing values across countries.

### How many results can one search return?

The actor keeps paging until your `maxItems` cap or the end of the result set. SimplyHired serves roughly 900–1,000 listings per search, even when it reports a larger match count. For more than that, turn on `maximizeCoverage`: the actor splits the search into facet sub-searches (employment types and developer skills), merges them and removes duplicates — in testing this returned 2.3x more unique jobs than a single search. You can also split by date window, radius, pay floor, or nearby locations yourself and combine the runs.

### What do the link fields point to?

`applicationUrl` is SimplyHired's own apply link for the posting — the same link the Apply button on the listing page uses. Open it from the job listing (or with an active SimplyHired session); it is not a direct employer URL, and the employer's own application page is reached by following that button in your browser. `companyPageUrl` is present only when SimplyHired includes an employer page for the posting: turn on `enrichCompanyPages` to look the employer up by name instead, and the actor adds the link (plus the employer's address, size, revenue band and industry) only when the page's own company name matches the listing's employer — mismatches and missing pages are omitted.

### Why is `employmentType` in English while the listing is in another language?

`employmentType` reports SimplyHired's own employment-type taxonomy (Full-time, Permanent, Contract, …), which the site uses across all its markets. `employmentTypes` also includes the listing's localized labels (for example "Festanstellung" or "CDI") when the posting provides them.

### Can I mix several keywords or countries in one run?

Yes. Add multiple `searchQueries`, or pass several `searchUrls` from different country markets — each is walked separately and results are deduplicated by `jobId`.

### Do I need an account, cookies, or a proxy?

No. The actor reads public pages only and connects on its own. Fill in the input and run it.

### How fresh is the data?

Each run queries SimplyHired live, so the records reflect the state of the site at run time. Schedule the actor if you want recurring snapshots.
