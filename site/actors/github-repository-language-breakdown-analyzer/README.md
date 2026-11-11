# GitHub Repository Language Breakdown Analyzer

GitHub shows one primary-language badge per repository, which hides the real mix of code inside. This actor reads GitHub's public REST API and returns each repository with byte counts and percentage shares for every detected language, plus optional ranked contributors, weekly commit statistics, and release history. Pick repositories one at a time, paste a list, sweep an organization or user account, or discover them by keyword or topic. No API token, login, or proxy is needed.

## What this actor does

- **Six ways to choose repositories:** one repository, a pasted list, an organization, a user account, a GitHub keyword search, or a topic search
- **Language detail:** byte counts per language beside the existing percentage breakdown, with the primary language and total analyzed bytes
- **Contributor rankings:** top contributors with public profile links and contribution counts, capped by your chosen limit
- **Commit statistics:** 52-week commit activity, all/owner weekly participation, weekly additions and deletions, and the weekday/hour punch card
- **Release history:** recent releases with tag, date, asset count, and total asset downloads
- **Discovery controls:** primary-language filter, fork and archive filters, sort field, sort direction, and a hard result cap
- Empty values are omitted from each record, so a field only appears when GitHub actually reports it

## Output per repository

- `repoId`, `repoName`, `fullName` — numeric ID, short name, and `owner/repo`
- `owner`, `ownerId`, `ownerType`, `ownerUrl`, `ownerAvatarUrl` — owner identity and public links
- `description` — repository description, capped at 300 characters
- `homepage`, `repoUrl`, `cloneUrl`, `gitUrl`, `sshUrl` — project and clone links
- `language` — GitHub's single primary-language label
- `languageBytes` — bytes of code per detected language, largest first
- `languagesBreakdown` — percentage share per language, rounded to two decimals
- `languageCount`, `totalLanguageBytes` — number of detected languages and total bytes analyzed
- `stars`, `forks`, `watchers`, `networkCount`, `openIssues` — popularity and issue counts; `watchers` is the real subscriber count when GitHub includes it
- `sizeKb`, `topics[]`, `topicsCount` — repository size and topic tags
- `defaultBranch`, `visibility` — default branch and public visibility
- `isForked`, `isArchived`, `isDisabled`, `isTemplate`, `isPrivate`, `allowForking` — repository state flags
- `hasIssues`, `hasProjects`, `hasWiki`, `hasPages`, `hasDownloads`, `hasDiscussions`, `hasPullRequests` — enabled feature flags
- `createdAt`, `updatedAt`, `pushedAt` — ISO timestamps for creation, metadata update, and last push
- `license`, `licenseSpdx`, `licenseKey` — declared license name, SPDX identifier, and key
- `score` — GitHub search relevance score, present in keyword and topic results
- `contributors[]` — ranked contributors with `login`, `name`, `email`, `contributorId`, `type`, `contributions`, `avatarUrl`, `profileUrl`, and `isAnonymous` for anonymous entries
- `contributorCount` — number of contributor rows returned
- `commitActivity[]` — 52 weekly rows with `weekStart`, `total`, and seven daily counts
- `commitActivityTotalLastYear` — total commits in the activity window
- `weeklyParticipation` — `all` and `owner` weekly commit counts
- `participationTotalLastYear` — sum of the all-contributor participation series
- `codeFrequency[]` — weekly `additions` and `deletions`
- `punchCard[]` — `day`, `hour`, and `commits` slots
- `releases[]` — recent releases with `tagName`, `name`, `publishedAt`, `createdAt`, `url`, `isDraft`, `isPrerelease`, `author`, `assetCount`, and `downloadCount`
- `releaseCount`, `latestRelease` — number of releases returned and the newest one
- `recordType` — always `github_repo`
- `scrapedAt` — UTC timestamp when the record was collected

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `orgRepos` | `repoDetail`, `repoList`, `orgRepos`, `userRepos`, `search`, or `topicRepos` |
| `repoFullName` | string | – | Repository for `repoDetail`; accepts `owner/repo` or a GitHub URL |
| `repoFullNames` | array | – | Repositories for `repoList`; accepts names and URLs |
| `org` | string | `microsoft` | Organization login for `orgRepos` |
| `username` | string | – | User login for `userRepos` |
| `searchQuery` | string | `stars:>1000` | Keyword or qualifier query for `search` |
| `topic` | string | `machine-learning` | Topic slug for `topicRepos` |
| `language` | string | – | Keep repositories whose primary language matches this name; applies to every mode, including named repositories |
| `includeForks` | boolean | `false` | Include forks in discovery results |
| `includeArchived` | boolean | `true` | Include archived repositories in discovery results |
| `sortBy` | string | `bestMatch` | `bestMatch`, `stars`, `forks`, `updated`, `created`, `pushed`, or `fullName` |
| `sortOrder` | string | `desc` | `desc` or `asc` |
| `maxItems` | integer | `20` | Hard cap on repository records (1–1000) |
| `includeLanguages` | boolean | `true` | Fetch language bytes and percentages |
| `includeContributors` | boolean | `false` | Fetch ranked contributors |
| `contributorsLimit` | integer | `10` | Contributors to keep per repository (1–100) |
| `includeCommitStats` | boolean | `false` | Fetch commit activity, participation, code frequency, and punch card |
| `includeReleases` | boolean | `false` | Fetch recent releases |
| `releasesLimit` | integer | `5` | Releases to keep per repository (1–30) |

### Example: one repository with every enrichment

```json
{
  "mode": "repoDetail",
  "repoFullName": "microsoft/vscode",
  "includeLanguages": true,
  "includeContributors": true,
  "contributorsLimit": 10,
  "includeCommitStats": true,
  "includeReleases": true,
  "releasesLimit": 5
}
```

### Example: a TypeScript-heavy organization sweep

```json
{
  "mode": "orgRepos",
  "org": "vercel",
  "language": "TypeScript",
  "includeForks": false,
  "includeArchived": false,
  "sortBy": "stars",
  "maxItems": 50
}
```

### Example: discover popular Python repositories

```json
{
  "mode": "search",
  "searchQuery": "web scraping stars:>5000",
  "language": "Python",
  "sortBy": "stars",
  "sortOrder": "desc",
  "maxItems": 30,
  "includeLanguages": true
}
```

### Example: compare a pasted repository list

```json
{
  "mode": "repoList",
  "repoFullNames": [
    "facebook/react",
    "https://github.com/vercel/next.js",
    "git@github.com:sveltejs/svelte.git"
  ],
  "includeLanguages": true,
  "includeContributors": true,
  "contributorsLimit": 5,
  "sortBy": "stars",
  "sortOrder": "desc"
}
```

## Use cases

- **Tech due diligence** — check what languages an acquisition target or vendor actually ships, not just its marketing page
- **Engineering benchmarking** — compare language mix, contributor depth, and commit cadence across competitor organizations
- **Open-source health checks** — review release rhythm, weekly participation, and top contributor concentration before adopting a dependency
- **Developer research** — find repositories in a niche by topic or keyword and rank them by stars or forks
- **Portfolio monitoring** — paste a watchlist of repositories and track stars, forks, language drift, and releases over time
- **License audits** — scan an organization's repositories for declared licenses and SPDX identifiers

## FAQ

**Do I need a GitHub token?** No. The actor uses GitHub's public endpoints without authentication. GitHub allows 60 unauthenticated core requests per hour per IP and 10 search requests per minute. When that budget runs out, the actor returns the repository records it already collected and tells you which enrichment was skipped.

**Why are some language fields missing?** Repositories with no analyzed code — documentation collections, awesome lists, empty repositories — genuinely have no language bytes on GitHub. The field is omitted rather than filled with a zero or a guess.

**What counts as a contributor?** GitHub's ranked contributor list groups commits by linked GitHub account and includes anonymous author entries when they are the only match. The list is capped by `contributorsLimit`, so `contributorCount` is the number returned, not necessarily every person who ever committed.

**Why did a big repository return no code-frequency data?** GitHub refuses code-frequency statistics for repositories with more than 10,000 commits. Other commit statistics still return when available.

**Why is `commitActivity` sometimes absent?** GitHub computes statistics lazily and can return a temporary "accepted" response for a repository it has not analyzed recently. The actor retries twice with a short delay; if the data is still pending it omits the field instead of fabricating it.

**What happens with more than 1,000 search results?** GitHub's search API stops at 1,000 matches. The actor reports the cap in the run status; narrow the query, split it by date or language, or use the organization/user modes, which paginate the full repository list.

**Are forks included by default?** No. Discovery modes skip forks unless `includeForks` is on. When you request a specific repository by name or URL, that repository is returned even if it is a fork, because you named it explicitly. The primary-language filter still applies to named repositories.

**Why does sorting by stars sometimes fall back to recently updated?** GitHub's organization and user listing endpoints sort by creation, update, push, or name, but not stars or forks. For those two sort options the actor switches to GitHub's repository search, which supports star and fork ranking and is capped at 1,000 results. Keyword-only sort choices that search does not support fall back to recently updated, and the run status says so.

**Can I fetch private repositories?** No. Only public repositories and public metadata are reachable without credentials.

**Is this affiliated with GitHub?** No. This is an independent, third-party actor built on GitHub's public REST API; it is not operated by or affiliated with GitHub, Inc.
