# Y Combinator Startup Directory with Founders & Jobs

Scrape **Y Combinator's startup directory** — all 6,337 companies across every batch since 2005 — with founders, open jobs, Launch YC posts, news and company media. Search by keyword, batch, industry, region, status or tag, or fetch specific companies by URL or slug. No login, no API key, no proxy.

## What this actor does

- **Two modes:** `search` for the full directory or filtered slices, and `byCompany` for a list of companies you already know.
- **Complete coverage:** all 50 YC batches and 6,337 companies, with no silent 1,000-row ceiling.
- **Company-first records:** batch, stage, status, industries, tags, badges, descriptions, locations, logo, socials, partner, photos, Launch YC posts, news and application answers.
- **Founders and jobs:** founder name, title, bio, LinkedIn, X, avatar and contactability flag; open jobs with salary and equity ranges, experience, visa, skills and optional full descriptions.
- **Empty fields are omitted** — no `null`, empty strings or placeholder values anywhere in a record.

## Output per company

- `type` — always `yc_company`
- `id`, `slug`, `url`, `name` — identity and YC profile link
- `companyId` — YC's numeric company ID
- `batch`, `batchName` — short code (e.g. `S24`) and full name (e.g. `Summer 2024`)
- `status` — `Active`, `Inactive`, `Acquired`, `Public` or `Not YC`
- `stage` — `Early` or `Growth`, when YC publishes it
- `industry`, `subIndustry`, `industries[]`, `tags[]`, `badges[]`
- `topCompany`, `isHiring`, `nonprofit` — present when YC flags the company
- `highlightBlackFounders`, `highlightWomenFounders`, `highlightHispanicFounders` — present when YC flags the company
- `shortDescription`, `longDescription`, `pitch`
- `website`, `logo`, `logoThumb`
- `foundingYear`, `launchedAt` (Unix timestamp), `formerNames[]`
- `teamSize` — omitted when YC reports 0 (unknown)
- `location`, `allLocations[]`, `city`, `country`, `regions[]`
- `linkedin`, `twitter`, `facebook`, `crunchbase`, `github`, `wellfound`
- `primaryPartner` — `{name, url, avatar}`
- `ycdcUrl`
- `photos[]` — public company photo URLs
- `videos[]` — `{title, videoUrl, libraryUrl, date}`
- `news[]` — `{title, url, date}`
- `launches[]` — `{id, title, tagline, url, createdAt, votes}` plus `body` when launch bodies are enabled
- `questionsAndAnswers[]` — `{question, answer}`
- `founders[]` — `{id, name, title, bio, avatar, linkedin, twitter, hasEmail, email, isActive, profileUrl}`; with founder profiles enabled, also `currentTitle`, `batches`, `previousRoles`, `schools` and extra social links
- `founderCount`
- `openJobs[]` — `{id, title, url, applyUrl, location, remote, type, role, roleSpecificType, prettyRole, minExperience, salaryRange, salaryMin, salaryMax, salaryCurrency, equityRange, equity, skills, visa, askUs, postedAgo}`; with job descriptions enabled, also `description`, `interviewProcess` and `hiringManager`
- `jobCount`
- `demoDayVideo`, `demoDayVideoUrl`, `applicationVideo`, `applicationVideoUrl`
- `kind`, `isCompanyHiring`, `topCompanyBadge`, `ycdcBadgeName`, `teamHighlights` — retained legacy fields, only when the source still supplies them
- `scrapedAt` — UTC timestamp of the scrape

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `search` | `search` filters the directory; `byCompany` fetches `companyUrls` |
| `searchQuery` | string | – | Free-text relevance search (mode=search) |
| `batch` | string | `any` | `any` or a YC batch code such as `S24`, `P25` or `F26` |
| `industry` | string | `any` | Exact YC industry facet |
| `region` | string | `any` | Exact YC region facet, including `Remote` variants |
| `status` | string | `any` | `any`, `Active`, `Inactive`, `Acquired` or `Public` |
| `tags` | array | `[]` | Exact YC tags; any one match is enough |
| `isHiring` | boolean | `false` | Only companies with YC's Is Hiring badge |
| `topCompany` | boolean | `false` | Only companies with YC's Top Company badge |
| `nonprofit` | boolean | `false` | Only companies with YC's Nonprofit badge |
| `highlightBlackFounders` | boolean | `false` | Only companies YC flags as having Black founders |
| `highlightWomenFounders` | boolean | `false` | Only companies YC flags as having women founders |
| `highlightHispanicFounders` | boolean | `false` | Only companies YC flags as having Hispanic or Latino founders |
| `minTeamSize` | integer | – | Drop companies below this employee count |
| `maxTeamSize` | integer | – | Drop companies above this employee count |
| `sortBy` | string | `default` | `default` directory order or `launchDate` (newest first) |
| `companyUrls` | array | `[]` | Full YC URLs, `/companies/<slug>` paths or bare slugs (mode=byCompany) |
| `scrapeFounders` | boolean | `true` | Add founders from the company page |
| `scrapeOpenJobs` | boolean | `true` | Add open jobs from the company page |
| `includeJobDescriptions` | boolean | `false` | Add full job descriptions, interview process and hiring manager |
| `includeLaunches` | boolean | `true` | Add Launch YC post metadata |
| `includeLaunchBodies` | boolean | `false` | Add the full text of each Launch YC post |
| `includeNews` | boolean | `false` | Add news items linked from the company page |
| `includeFounderProfiles` | boolean | `false` | Add previous roles, schools and current title per founder |
| `maxItems` | integer | `50` | Stop after this many companies (1–7,000), applied after filters |

### Example: a batch's fintech companies with founders and job details

```json
{
  "mode": "search",
  "batch": "S24",
  "industry": "Fintech",
  "scrapeFounders": true,
  "scrapeOpenJobs": true,
  "includeJobDescriptions": true,
  "maxItems": 100
}
```

### Example: export the complete directory

```json
{
  "mode": "search",
  "scrapeFounders": true,
  "scrapeOpenJobs": false,
  "includeLaunches": true,
  "maxItems": 7000
}
```

### Example: look up specific companies

```json
{
  "mode": "byCompany",
  "companyUrls": [
    "https://www.ycombinator.com/companies/airbnb",
    "/companies/stripe",
    "doordash"
  ],
  "scrapeFounders": true,
  "scrapeOpenJobs": true,
  "includeLaunchBodies": true
}
```

### Example: top hiring companies in Europe, newest launches first

```json
{
  "mode": "search",
  "region": "Europe",
  "topCompany": true,
  "isHiring": true,
  "minTeamSize": 50,
  "sortBy": "launchDate",
  "scrapeFounders": true,
  "maxItems": 200
}
```

## Use cases

- **Lead generation** — build batch, industry or region lists with founder names, titles and LinkedIn profiles.
- **Recruiting** — pull open YC roles with salary ranges, equity, experience requirements and full descriptions.
- **Market mapping** — compare companies across batches, industries, regions, team sizes and stage.
- **Competitive research** — track launches, news and product positioning from the directory.
- **Diversity sourcing** — use the founder highlight badges YC itself publishes.
- **Data enrichment** — fetch known companies by URL or slug and append founders, jobs and media.

## FAQ

**Do I need a login or API key?**
No. The directory, company pages, founder profiles and job pages are public.

**How many companies are in the directory?**
About 6,337 companies across 50 batches, growing with each new batch. A single run can export up to 7,000 companies.

**Why is there no revenue, ARR or funding data?**
Y Combinator does not publish revenue, valuation or funding amounts in the directory, company pages or job pages. The actor does not infer or fabricate them.

**Why are there no founder email addresses?**
YC publishes only a contactability flag, surfaced as `hasEmail`. The actor does not guess or verify email addresses.

**Why are some founders, jobs, launches or news items missing?**
Those sections are published per company. When a company has none, the field is omitted rather than filled with an empty value. Companies with a hiring badge can still have no public job postings.

**Where do `stage`, `launchedAt` and `formerNames` come from?**
From YC's own directory metadata. When that metadata is unavailable for a run, those fields are omitted and the run continues.

**Why is `teamSize` missing for some companies?**
Some companies report 0 employees, which means unknown. A zero is not a real team size, so it is omitted and size filters exclude it.

**Do I need a proxy?**
No. The actor reads public pages directly without a proxy or browser.

**Is this affiliated with Y Combinator?**
No. This is an independent, third-party actor that reads Y Combinator's public directory and is not operated by or affiliated with Y Combinator.
