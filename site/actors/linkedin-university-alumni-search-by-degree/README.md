# LinkedIn University Alumni Insights & Profiles

See what a university's alumni community looks like on LinkedIn, then find individual public alumni profiles that match a degree, field of study, graduation year, employer, or location — without a LinkedIn login, cookies, or proxy setup.

## What this actor does

- **School insights mode:** for any LinkedIn school page, returns the public alumni count plus the top 10 employers, functions, and fields of study, with raw counts and LinkedIn's displayed share where the page shows one.
- **One-facet drill-down:** narrow a school's insights by a single field of study, function, or company using the facets LinkedIn exposes publicly.
- **Alumni profiles mode:** searches public LinkedIn profiles indexed for a school, opens each public profile, verifies the school in its education history, and returns name, profile URL, location, current company, photo, about, and education dates.
- **Degree, field, and year filters:** degree and field are best-effort because LinkedIn hides them on most signed-out profiles; graduation year is read from public education dates. Every value is source-flagged and never guessed.
- **School profile details:** optionally add the school's website, founding year, staff size, specialties, and public employee count.

## Output per school insight

- `recordType: "schoolInsight"`
- `schoolName`, `schoolSlug`, `schoolUrl`, `schoolId`
- `alumniCount` — people on LinkedIn who list the school
- `topEmployers[]`, `topFunctions[]`, `topFieldsOfStudy[]` — each with `name`, `count`, `sharePercent`, `facetId`, and `linkedinUrl`
- `industry`, `location`, `followersCount`, `description`
- `insightDimension`, `insightValue`, `insightCount` — present when a facet was applied
- `website`, `founded`, `schoolSize`, `headquarters`, `schoolType`, `specialties[]`, `employeeCount` — when school profile details are enabled
- `scrapedAt`

`sharePercent` is the percentage LinkedIn displays; it uses a different denominator from `alumniCount`, so read the label as-is.

## Output per alumnus

- `recordType: "alumnus"`
- `name`, `profileUrl`, `location`, `about`, `followersText`, `followersCount`
- `currentCompany` — public current employer with `name`, `slug`, and `url` when available
- `profilePicture`, `profileImageUrl` — public photo when LinkedIn returns a real one
- `schoolName`, `schoolUrl`, `schoolSlug`, `schoolId`
- `education[]` — each public education entry with school, URL, degree, field of study, start/end year, and current-student flag when shown
- `degree`, `fieldOfStudy`, `graduationYear` — top-level values for the matched school entry
- `degreeSource`, `fieldOfStudySource`, `graduationYearSource` — `profile` or `search_snippet`
- `headline`, `headlineSource` — only when a real public role is visible
- `discoveredVia`, `searchEngine`, `searchQuery`, `searchRank`
- `scrapedAt`

Fields LinkedIn does not expose to signed-out visitors (`currentTitle`, connection degree, verified status, match snippets) are omitted rather than filled with placeholders. Empty values are omitted everywhere.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `schoolInsights` | `schoolInsights` or `alumniProfiles` |
| `schoolUrls` | array | – | LinkedIn school URLs or bare slugs, e.g. `https://www.linkedin.com/school/mit/` or `mit` |
| `schoolNames` | array | – | Plain school names, e.g. `MIT`, `Harvard University` |
| `maxSchools` | integer | `10` | Maximum schools per run (1–100) |
| `maxAlumniPerSchool` | integer | `50` | Maximum verified profiles per school in alumni mode (1–200; larger legacy values are clamped to 200) |
| `degree` | string | – | `Bachelor`, `Master`, `MBA`, `PhD`, or `Associate` (alumni mode; best-effort) |
| `fieldOfStudy` | string | – | Insights: field facet name or ID. Alumni: field search hint |
| `currentFunction` | string | – | Insights only: function facet name or ID |
| `currentCompany` | string | – | Insights: company facet. Alumni: strict current-employer filter |
| `currentLocation` | string | – | Alumni only: strict location filter |
| `graduationYearRange` | object | – | Alumni only: inclusive `{start, end}` year range |
| `includeSchoolProfile` | boolean | `true` | Insights only: add one overview request for school profile details |

At least one of `schoolUrls` or `schoolNames` is required. Use `maxSchools` and `maxAlumniPerSchool` to keep runs small.

### Example: a school's public alumni insights

```json
{
  "mode": "schoolInsights",
  "schoolUrls": ["https://www.linkedin.com/school/mit/"],
  "maxSchools": 1
}
```

### Example: only computer-science alumni in a field

```json
{
  "mode": "schoolInsights",
  "schoolUrls": ["https://www.linkedin.com/school/stanford-university/"],
  "fieldOfStudy": "Computer Science",
  "maxSchools": 1
}
```

### Example: public MBA alumni profiles

```json
{
  "mode": "alumniProfiles",
  "schoolNames": ["Massachusetts Institute of Technology"],
  "degree": "MBA",
  "graduationYearRange": {"start": 2015, "end": 2022},
  "maxAlumniPerSchool": 25
}
```

### Example: alumni now at a specific company

```json
{
  "mode": "alumniProfiles",
  "schoolUrls": ["https://www.linkedin.com/school/oxforduni/"],
  "currentCompany": "Google",
  "maxAlumniPerSchool": 10
}
```

## Use cases

- **University recruiting** — find public profiles of graduates from target schools and programs
- **Tracer studies and alumni relations** — see the employers, functions, and fields a school's alumni report publicly
- **Talent-market research** — compare public alumni outcomes across universities and fields
- **Sales and partnerships** — build school-filtered lists of public professional profiles
- **Academic research** — study publicly stated degree, field, and graduation-year patterns

## FAQ

### Is this affiliated with LinkedIn?

No. This is an independent, third-party actor that reads public LinkedIn pages as a signed-out visitor. It is not operated by or affiliated with LinkedIn.

### Do I need a LinkedIn login, cookie, or proxy?

No. The actor uses only public pages and needs no account, cookie, or proxy configuration.

### Why do some alumni profiles have no degree or field of study?

LinkedIn hides degree and field on most signed-out profile pages. Across public samples, a degree is visible for roughly one in six profiles and a field of study for about one in twenty. The actor fills them only when the public profile or the search result snippet shows them, and marks the source. It never guesses a degree from a job title.

### Where do the individual alumni profiles come from?

Public profiles are discovered through Yahoo's public search index (`site:linkedin.com/in`), then opened as signed-out LinkedIn profile pages. Yahoo's coverage and ranking vary, so treat the result as a bounded sample; the actor stops at `maxAlumniPerSchool` or when the public source runs out.

### Why is the graduation year missing for some alumni?

About three quarters of public education entries include dates. When LinkedIn does not show a date, the field is omitted. Setting `graduationYearRange` drops profiles without a public year.

### Why can I filter by only one facet in school insights?

LinkedIn's public school page exposes one facet per request. A second facet redirects to a login wall, so the actor applies the first one and tells you in the run status which one it used.

### How many alumni can I get for one school?

A discovery run is a sample, not a census. Public search indexes and LinkedIn's guest pages limit how many profiles are reachable; the actor stops at `maxAlumniPerSchool` or when the public source is exhausted. Large schools have hundreds of thousands of alumni, so use filters to target the slice you need.

### Why does the alumni count not match the percentages?

`alumniCount` is the total number of people who list the school. The displayed percentage uses a different denominator (people with that outcome visible), so the actor preserves LinkedIn's displayed share instead of recomputing it.

### Can I get email addresses, phone numbers, or full work history?

No. Those are not public on signed-out LinkedIn profiles, so this actor does not return them.

### Does it work for schools outside the United States?

Yes. School URLs, slugs, names, and profiles in non-English markets are supported; unicode school names and slugs are handled.
