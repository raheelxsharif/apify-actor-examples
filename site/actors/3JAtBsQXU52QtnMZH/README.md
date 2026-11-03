# Company Intent Scorer: Hiring, Funding & News Signals

Turn a list of target accounts into a ranked buying-signal report. For every company this actor checks public job boards, SEC Form D filings and funding coverage, entity-filtered news, executive changes, layoff risk and the company homepage, then returns one graded record with the evidence behind the score. No cookies, logins, API keys or proxy setup are needed.

## What this actor does

- **Scores and ranks every company** on a 0–100 intent score with an A–D grade, `priorityFlag`, `topSignal`, `topSignals[]` and a plain-English `rationale`
- **Finds real hiring** on nine public applicant-tracking systems, including Workday, with role counts, 7/30/90-day velocity, function/seniority mix, remote counts and locations
- **Finds real funding** from SEC Form D filings and entity-matched press, with amount, round, valuation, date and confidence
- **Filters news to the company**, then derives news momentum, executive changes and layoff risk from the same matched headlines
- **Adds a technology stack** as enrichment and can check for one target technology without changing the intent score
- **Monitors changes** between runs: new signals, resolved signals, score movement and whether a company changed at all
- **Omits empty fields** at every depth instead of returning nulls

## Output per company

Every record includes:

- `companyName` — company name as provided
- `domain` — verified primary domain, when one can be confirmed
- `companyUrl` — homepage URL built from the verified domain
- `careersUrl` — careers page or public job-board URL when found
- `ats` — job-board provider when a board was found
- `grade` — A, B, C or D
- `score` — intent score from 0 to 100
- `priorityFlag` — true for grade A, funding in the last 180 days, or an executive change in the last 30 days
- `topSignal` — one-line summary of the strongest signal
- `topSignals[]` — up to three strongest signals with `type`, `label`, `summary`, `score`, `maxScore`, `latestDate`, `source` and `url`
- `rationale` — why the company received its grade, using the real counts and dates
- `runTimestamp` — ISO 8601 run timestamp
- `errors[]` — per-source notes when a signal could not be fetched or a board was not found
- `targetTechnology` — when requested: `{ name, present, category?, confidence? }`
- `monitor` — in monitor mode: `previousRunAt`, `newSignals[]`, `resolvedSignals[]`, `scoreDelta` and `changed`

With `outputFormat: "full"` the record adds:

- `scoreBreakdown` — normalized point contributions for `hiring`, `funding`, `news` and `execChanges`, plus `riskAdjustment`
- `hiringSignal` — `openRoles`, `postedLast7Days`, `postedLast30Days`, `postedLast90Days`, `remoteRoles`, `functionMix`, `seniorityMix`, `departmentMix`, `locations`, `newestPostingDate`, legacy `totalOpenRoles`, `rolesLast30Days`, `hasExecutiveRoles`, `hasSalesRoles`, `roleMix`, and sample titles/jobs when raw evidence is enabled
- `fundingSignal` — `amountUsd`, `roundType`, `announcedDate`, `daysSinceRound`, `valuationUsd`, `source`, `sourceUrl`, `headline`, `confidence`, `secFilingUrl`, `investors[]`
- `newsSignal` — `mentionsLast7Days`, `articleCount30d`, `mentionsLast90Days`, positive/negative/neutral counts, `topicTags`, `topSourceDomains`, and top headlines when raw evidence is enabled
- `techStackSignal` — `technologies[]` with name, category, confidence and evidence, plus `categories`, `detectedCount`, `homepageUrl` and the target-technology result
- `execChangeSignal` — `changes[]` with title, person, level, date, source and URL
- `riskSignal` — `events[]` with confirmed/hedged layoff items
- `signals[]` — normalized evidence rows for every enabled signal

If every enabled source fails for a company, the record instead contains `type: "company_error"`, `companyName`, `errors[]` and `runTimestamp`.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `score` | `score` or `monitor` |
| `companiesText` | string | – | Company names, one per line |
| `companies` | array | – | Advanced list of objects (or plain strings); accepts `companyName`/`name`, `domain`, `careersUrl`, `ats`, `atsToken`, `disambiguator` |
| `signals` | array | `["hiring","funding","news","execChanges","layoffs","techStack"]` | `hiring`, `funding`, `news`, `execChanges`, `layoffs`, `techStack` |
| `lookbackDays` | integer | `90` | Funding, news and executive-change window (7–365) |
| `minScore` | integer | `0` | Drop scored companies below this score |
| `targetTechnology` | string | – | Optional technology gap check, e.g. `HubSpot` |
| `includeRawSignals` | boolean | `false` | In full output, add sample job titles and top headlines |
| `outputFormat` | string | `compact` | `compact` ranking or `full` evidence detail |
| `maxCompanies` | integer | `100` | Hard cap per run (1–1000) |
| `maxConcurrency` | integer | `3` | Companies processed in parallel (1–10) |
| `monitorKey` | string | – | Tracking history name for `mode: "monitor"` |
| `emitUnchanged` | boolean | `false` | In monitor mode, also emit companies with no change |

### Example: score a short target list

```json
{
  "mode": "score",
  "companiesText": "Figma\nNotion\nLinear",
  "outputFormat": "compact"
}
```

### Example: weekly account monitoring

```json
{
  "mode": "monitor",
  "companiesText": "Figma\nNotion\nLinear\nRamp\nDatabricks",
  "monitorKey": "q4-enterprise-targets",
  "minScore": 40,
  "maxCompanies": 200
}
```

### Example: full evidence with a technology gap check

```json
{
  "mode": "score",
  "companiesText": "Databricks\nSupabase\nCloudflare",
  "outputFormat": "full",
  "includeRawSignals": true,
  "targetTechnology": "Marketo",
  "lookbackDays": 120
}
```

### Example: known domains and job boards

```json
{
  "mode": "score",
  "companies": [
    { "companyName": "Notion", "domain": "notion.so", "disambiguator": "workspace" },
    { "companyName": "Linear", "careersUrl": "https://jobs.ashbyhq.com/linear" },
    { "companyName": "Scale AI", "ats": "greenhouse", "atsToken": "scaleai" }
  ],
  "outputFormat": "full"
}
```

## Use cases

- **SDR and AE prioritization** — sort a territory by intent grade before a prospecting block
- **Account-based marketing** — trigger plays when a target starts hiring, raises a round or changes leadership
- **RevOps hygiene** — re-score dormant accounts and filter out anything below the score your team acts on
- **Competitive and market research** — compare hiring pace, funding events and technology choices across a market
- **Customer expansion** — watch existing accounts for new funding, new executives and hiring surges
- **Weekly change monitoring** — use `mode: "monitor"` to see only the accounts that changed since the last run

## FAQ

**What sources does the score use?**
Public applicant-tracking boards for hiring, SEC Form D filings and press coverage for funding, Google News RSS for entity-matched news, and the company's public homepage for technology detection.

**How is the score calculated?**
Hiring contributes up to 40 points, funding up to 35, news up to 15 and executive changes up to 10. If you disable a scored signal, the remaining weights are normalized to 100. A confirmed company-specific layoff in the last 90 days subtracts up to 10 points; risk can never raise the score. Grade A is 70+, B is 45–69, C is 20–44 and D is below 20.

**Why is funding missing for some companies?**
The actor reports only funding it can verify from an SEC Form D or an entity-matched press headline with a real amount or round. Private companies without a recent filing or clear coverage are left without a funding signal rather than guessed at.

**Why does a company have no hiring signal?**
Only public job boards are used. If the company's careers site does not expose one of the supported boards, the actor omits the hiring signal and adds a note instead of counting unrelated jobs.

**Why is news missing or smaller than a web search?**
Headlines must contain the company name as the subject, and tracker pages, listicles, homonyms and another-entity mentions are removed. Add a `disambiguator` in the advanced list when a company name is a common word.

**Do I need an API key or a proxy?**
No. The actor uses public, cookieless sources. Just fill in the input and run it.

**How does monitor mode remember the previous run?**
It stores a compact snapshot under your `monitorKey`. Use the same key on every scheduled run; a new key starts a fresh baseline. `emitUnchanged` controls whether unchanged companies appear too.

**What is the difference between compact and full output?**
Compact returns the ranking fields and top signals. Full adds the score breakdown and the detailed hiring, funding, news, technology, executive-change and risk objects.

**Why are some Workday posting dates approximate?**
Workday normally shows relative dates such as "Posted 5 Days Ago" or "30+ Days Ago". The actor maps those to dates and marks the hiring signal as approximate when a board uses them.

**Is this affiliated with the job boards, SEC or news publishers?**
No. This is an independent actor that reads public information; it is not affiliated with or endorsed by those services.
