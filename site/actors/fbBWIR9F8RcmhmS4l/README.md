# Y Combinator Startup Directory with Founders & Jobs

Pull structured records from Y Combinator's public company directory — every batch since 2005 — filtered by batch, industry, status, region, or free-text search, with each company's founders and open job postings attached.

## What this actor does

- Queries Y Combinator's public directory search and returns one record per matching company.
- Filters by batch code, industry, operating status, free-text query, region, and tags, and can restrict results to companies flagged with underrepresented founder signals when YC itself publishes that flag.
- Optionally fetches each company's own profile page to attach its founders (name, title, bio, social links) and current open job postings (title, location, salary range, equity, skills, visa support).
- Caps the run at a configurable number of companies so you can scrape a handful for testing or the whole directory for a full export.
- When a filter combination matches zero companies, the actor emits a single status record rather than an empty, unexplained run; every other field a company doesn't have published is simply left out of its record.

## Output per company

- `type` — always `yc_company`
- `id`, `slug`, `url`, `name` — identity and profile link
- `kind` — `startup`, `non-profit`, or similar company kind
- `shortDescription`, `longDescription`, `pitch` — the company's one-liner, long description, and full pitch text when published
- `batch` — YC batch code (e.g. `S24`)
- `industry`, `subIndustry`, `tags` — YC's own category facets
- `status` — `Active`, `Inactive`, `Acquired`, or `Public`
- `stage` — funding stage (e.g. `Seed`, `Series A`), when published
- `location`, `allLocations`, `regions`
- `foundingYear`, `launchedAt` (unix timestamp), `teamSize`
- `website`, `linkedin`, `twitter`, `facebook`, `crunchbase`, `wellfound`, `github`
- `logo`, `logoThumb`, `demoDayVideo`
- `isHiring`, `isCompanyHiring`, `jobCount`
- `formerNames` — previous company names, when it has rebranded
- `topCompany`, `topCompanyBadge`, `ycdcBadgeName`, `nonprofit` — only present when YC flags them
- `questionsAndAnswers` — short founder Q&A blocks, each `{question, answer}`
- `teamHighlights` — blurbs about the team
- `highlightBlackFounders`, `highlightWomenFounders`, `highlightHispanicFounders` — present only when YC's directory explicitly flags the company
- `founders` — `{name, title, bio, email, linkedin, twitter, hackerNews, github, instagram}` per founder, when `scrapeFounders` is `true`; `email` is `true` when YC marks the founder as contactable (the actual address is never published)
- `openJobs` — `{title, url, applyUrl, location, remote, type, role, team, yearsExperience, salaryMin, salaryMax, compensationCurrency, equity, equityRange, skills, experience, visa, visaSupported, englishFluent}` per posting, when `scrapeOpenJobs` is `true`
- `scrapedAt` — ISO 8601 timestamp of when the record was scraped

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `directoryUrl` | string | `https://www.ycombinator.com/companies` | YC directory URL, optionally with its own query filters already in it |
| `query` | string | – | Free-text search matched against name, description, and tags |
| `batch` | string | `any` | YC batch code (e.g. `S24`, `W24`) or `any` |
| `industry` | string | – | Case-sensitive, exact-match industry facet (e.g. `B2B`, `Fintech`, `Healthcare`) |
| `status` | string | `any` | `any`, `Active`, `Inactive`, `Acquired`, or `Public` |
| `scrapeFounders` | boolean | `true` | Fetch founder details per company |
| `scrapeOpenJobs` | boolean | `true` | Fetch open job postings per company |
| `regions` | array | `[]` | Case-insensitive substring filter on company location |
| `tags` | array | `[]` | Case-insensitive filter against industry, sub-industry, and tags |
| `highlightBlackFounders` | boolean | `false` | Only return companies YC flags as having Black founders |
| `highlightWomenFounders` | boolean | `false` | Only return companies YC flags as having women founders |
| `highlightHispanicFounders` | boolean | `false` | Only return companies YC flags as having Hispanic/Latino founders |
| `maxItems` | integer | `3` | Maximum number of companies to return (1–5500) |

### Example: a batch's fintech companies with founders and jobs

```json
{
  "directoryUrl": "https://www.ycombinator.com/companies",
  "batch": "S24",
  "industry": "Fintech",
  "scrapeFounders": true,
  "scrapeOpenJobs": true,
  "maxItems": 100
}
```

### Example: free-text search across the whole directory

```json
{
  "directoryUrl": "https://www.ycombinator.com/companies",
  "query": "developer tools",
  "status": "Active",
  "maxItems": 50
}
```

### Example: lightweight export, profile fields only

```json
{
  "directoryUrl": "https://www.ycombinator.com/companies",
  "regions": ["San Francisco"],
  "scrapeFounders": false,
  "scrapeOpenJobs": false,
  "maxItems": 500
}
```

## Use cases

- **VC and startup research** — build a filtered dataset of companies by batch, industry, or stage
- **Recruiting** — pull current open roles across YC's portfolio with salary and equity ranges
- **Founder sourcing** — collect founder names and social links for a specific batch or industry
- **Competitive mapping** — track who else YC has backed in a given space or region
- **Diversity-focused sourcing** — surface companies YC itself has flagged with underrepresented founder signals

## FAQ

**Do I need a login or API key?**
No — the directory, company profiles, and job postings are all public.

**How many companies are in the directory?**
Roughly 5,000-plus across every batch since Y Combinator's founding, growing with each new batch; `maxItems` caps how many a single run returns (up to 5,500).

**Why do `founders` and `openJobs` sometimes come back empty on a run?**
They're only fetched when `scrapeFounders`/`scrapeOpenJobs` are `true`; turning them off skips the extra per-company page fetch for a faster run.

**Why does `jobCount` sometimes differ from what I see on the YC site?**
`jobCount` reflects exactly the open postings returned on the company's own profile page at scrape time, which is the same source the site's badge is built from.

**What do the `highlightBlackFounders`/`highlightWomenFounders`/`highlightHispanicFounders` filters depend on?**
They only match companies where YC's own directory has published that flag — the actor never infers or guesses these signals.

**What happens if my filters match no companies?**
The run still produces one status record saying nothing matched, instead of an empty, unexplained dataset.

**Can I pass my own pre-filtered directory URL?**
Yes — `directoryUrl` accepts any `ycombinator.com/companies` URL with its own query parameters already applied, which the actor layers its own filters on top of.

**Is this affiliated with Y Combinator?**
No. This is an independent, third-party actor that reads Y Combinator's public directory and is not operated by or affiliated with Y Combinator.
