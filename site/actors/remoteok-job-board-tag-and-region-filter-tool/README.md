# RemoteOK Job Board Tag & Region Filter Tool

Turn RemoteOK's job board into structured data filtered the way you actually search: by keyword, by one of its 240+ skill/category tags, by region, by salary range, or by how recently a job was posted. Each listing comes back with position, company, salary, tags, location, and a direct apply link, with the description cleaned of HTML markup.

## What this actor does

- **Five modes:**
  - `search` — free-text keyword filter over the latest feed (title, company, description, tags)
  - `byTag` — pull all jobs tagged with a single Remote OK tag (`python`, `design`, `marketing`, ...)
  - `byTags` — pull jobs across multiple tags and merge (jobs matching ANY of the tags)
  - `byUrls` — fetch jobs by direct Remote OK job page URLs
  - `latest` — full latest feed with no keyword/tag filter
- **Tag-based filtering** — type any Remote OK tag (e.g. `python`, `design`, `data-science`); 240+ tags are supported
- **Region filtering** — pick `worldwide` / `americas` / `europe` / `asia` / `africa` / `oceania` / `usOnly` from a dropdown; regions are inferred from each job's location string plus its `digital nomad` tag
- **Salary filtering** — `minSalary`, `maxSalary`, and `salaryOnly` (drop jobs without a published range)
- **Recency filtering** — `postedSinceDays` for "last N days"
- **Keyword filter** — extra `containsKeyword` substring over title/company/description/tags
- **HTML descriptions cleaned** — entities decoded, tags stripped, whitespace collapsed
- **Empty fields are omitted** from every record

## Output per job

- `jobId` — Remote OK job ID (e.g. `1131632`)
- `slug` — Remote OK URL slug
- `position` — job title
- `company` — company / employer name
- `location` — raw location text from Remote OK (e.g. `Dallas, TX`, `Worldwide`, `Berlin`)
- `region` — derived region bucket (`worldwide` / `americas` / `europe` / `asia` / `africa` / `oceania`)
- `salaryMin`, `salaryMax`, `salaryCurrency` — published USD salary range (when available)
- `tags` — lowercased, deduplicated, sorted skill / category tags
- `description` — plaintext job description (HTML stripped, capped at 8000 chars)
- `url` — canonical Remote OK job page URL
- `applyUrl` — direct apply URL (when different from canonical URL)
- `companyLogo` — company logo URL (when published)
- `postedAt` — ISO-8601 UTC posting timestamp
- `verified` — `true` if Remote OK has marked the listing verified
- `recordType: "job"`, `scrapedAt` — added by the actor

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `search` | `search` / `byTag` / `byTags` / `byUrls` / `latest` |
| `searchQuery` | string | `engineer` | Free-text query (mode=search) |
| `tag` | string | `python` | Single Remote OK tag (mode=byTag) — free-text, e.g. `python`, `design` |
| `tags` | array | – | Multiple Remote OK tags (mode=byTags) — any-of |
| `startUrls` | array | – | Direct Remote OK job URLs (mode=byUrls) |
| `region` | string | `any` | `any` / `worldwide` / `americas` / `europe` / `asia` / `africa` / `oceania` / `usOnly` — pick from dropdown |
| `minSalary` | int | – | Drop jobs whose max salary is below this value (USD) |
| `maxSalary` | int | – | Drop jobs whose min salary is above this value (USD) |
| `postedSinceDays` | int | – | Only emit jobs posted in the last N days |
| `salaryOnly` | bool | `false` | Drop jobs that do not publish a salary |
| `containsKeyword` | string | – | Additional substring filter |
| `maxItems` | int | `50` | Hard cap on emitted records (1–1000) |

### Example: senior remote Python jobs worldwide

```json
{
  "mode": "byTag",
  "tag": "python",
  "region": "worldwide",
  "containsKeyword": "senior",
  "maxItems": 50
}
```

### Example: latest design jobs over $100k

```json
{
  "mode": "byTag",
  "tag": "design",
  "minSalary": 100000,
  "salaryOnly": true,
  "maxItems": 30
}
```

### Example: pull a specific job by URL

```json
{
  "mode": "byUrls",
  "startUrls": [
    "https://remoteok.com/remote-jobs/remote-senior-fraud-risk-analyst-braviant-holdings-1131632"
  ]
}
```

### Example: any of multiple tags

```json
{
  "mode": "byTags",
  "tags": ["data-science", "machine-learning", "ai"],
  "postedSinceDays": 14,
  "maxItems": 100
}
```

## Use cases

- **Job aggregators** — bulk-ingest fresh remote jobs into your search / matching engine
- **Recruiter intelligence** — track who's hiring, in which tech stacks, with what salary ranges
- **Salary benchmarking** — measure compensation ranges for specific roles or stacks
- **Market research** — see which regions are publishing the most remote roles for your niche
- **Talent platforms** — power "trending remote roles" or "highest-paying remote tag" dashboards
- **Personal job hunt** — alert me when a Python role over $150k posts in Europe

## FAQ

**What's Remote OK?**  Remote OK is one of the largest remote job boards. It aggregates fully remote tech, design, marketing, and ops jobs from companies around the world. See [remoteok.com](https://remoteok.com).

**Do I need login or API key?**  No. Remote OK's job listings are public, so the actor runs without any authentication.

**How many jobs does the feed return?**  About 100 latest jobs at the global level, and varying counts per tag (typically 50–150 depending on the tag's popularity).

**What regions does region filtering support?**  `worldwide`, `americas`, `europe`, `asia`, `africa`, `oceania`, and `usOnly`. Remote OK encodes worldwide jobs via the `digital nomad` tag and the rest are inferred from each job's `location` string.

**Why is the salary range often missing?**  Most Remote OK postings don't publish salary. Roughly 10–15% include a range. Set `salaryOnly: true` if you only want priced postings.

**Can I get more than 100 jobs in a single run?**  Yes — combine `byTags` with multiple tags. The actor fetches each tag's feed and merges them, deduplicated by job ID. You can reach 500+ unique jobs across half a dozen tags.

**Are job descriptions formatted as HTML or plaintext?**  The actor strips HTML tags and decodes entities so the `description` field is clean plaintext. The original HTML is available on the job page if needed.

**What's the difference between `url` and `applyUrl`?**  `url` is the canonical Remote OK page; `applyUrl` is the company's direct apply link when distinct from the Remote OK page.

**How fresh is the data?**  Real-time — Remote OK publishes new postings as they're approved, and each run reflects whatever is live at that moment. Set `postedSinceDays: 1` to capture only the most recent.

**Why does `byUrls` mode sometimes return fewer records than I passed in?**  A job that has aged off Remote OK's current listings (typically after 1–2 weeks) can no longer be resolved by URL, since the actor only reads currently live postings.

**Is this affiliated with Remote OK?**  No. This is an independent, third-party actor that reads Remote OK's publicly available job listings and is not affiliated with or endorsed by Remote OK.
