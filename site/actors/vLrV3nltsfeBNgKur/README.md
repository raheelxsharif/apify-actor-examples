# GitHub Repository Language Breakdown Analyzer

Every GitHub repository mixes several programming languages under the hood, but the repo list and search pages only show a single "primary language" badge. This actor calls GitHub's public REST API to pull repository metadata for an entire organization, a keyword search, or one specific repo, and attaches a full percentage breakdown of every language used in each repository.

## What this actor does

- Lists every public repository belonging to a GitHub organization, newest-updated first
- Searches all of GitHub's public repositories by keyword, sorted by star count
- Fetches a single repository by its `owner/repo` full name
- Computes a `languagesBreakdown` percentage split (by bytes of code) for every repository it returns
- Empty fields are omitted from records — a field only appears when GitHub actually reports that data for the repo

## Output per repository

- `repoId` — GitHub's numeric repository ID
- `repoName` — repository name
- `owner` — repository owner's GitHub username/org
- `fullName` — `owner/repo` identifier
- `description` — repository description, truncated to 300 characters
- `language` — GitHub's single "primary language" label
- `languagesBreakdown` — object mapping each language used to its percentage share of the repo's code
- `stars` — star count
- `forks` — fork count
- `topics` — GitHub topic tags attached to the repo
- `defaultBranch` — default branch name
- `isForked` — `true` if the repository is a fork of another repo
- `openIssues` — open issue count
- `createdAt`, `updatedAt` — repository creation and last-update timestamps
- `license` — license name, when the repo declares one
- `repoUrl` — link to the repository on github.com
- `recordType` — always `github_repo`
- `scrapedAt` — UTC timestamp when the record was collected

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `orgRepos` | `orgRepos` to list an organization's public repos, `search` to run a keyword search across GitHub, or `repoDetail` for one specific repo |
| `searchQuery` | string | – | Keywords to search for (mode = `search`) |
| `org` | string | `microsoft` | GitHub organization name (mode = `orgRepos`) |
| `repoFullName` | string | – | Full repository name, e.g. `facebook/react` (mode = `repoDetail`) |
| `language` | string | – | Restrict results to repositories whose primary language matches this value |
| `maxItems` | integer | `20` | Maximum number of repository records to return (1–100) |

### Example: tech-stack sweep of an organization

```json
{
  "mode": "orgRepos",
  "org": "vercel",
  "maxItems": 50
}
```

### Example: top Python scraping projects by stars

```json
{
  "mode": "search",
  "searchQuery": "web scraper",
  "language": "Python",
  "maxItems": 30
}
```

### Example: single repository detail

```json
{
  "mode": "repoDetail",
  "repoFullName": "facebook/react"
}
```

## Use cases

- **Tech due diligence** — pull every repo in a target organization to see what languages and frameworks it actually runs on
- **Engineering benchmarking** — compare your org's language mix against a competitor's public repos
- **Open-source landscape research** — search a keyword like "LLM agent" and sort the results by stars to see what's trending
- **Dependency/stack audits** — check the `languagesBreakdown` of a specific repo before adopting it
- **License compliance checks** — scan an organization's repos for their declared `license` field

## FAQ

**Do I need a GitHub token?** No — this actor uses GitHub's public REST API endpoints, which work without authentication for the data this actor returns.

**What's the difference between `language` and `languagesBreakdown`?** `language` is GitHub's single-label "primary language" for the repo. `languagesBreakdown` is a full percentage split across every language detected in the repository's code, computed from GitHub's languages endpoint.

**Why does the `language` input filter sometimes return fewer results in `orgRepos` mode than expected?** It filters against each repo's primary `language` field client-side after fetching the organization's repos, so repositories where that language is only a minor part of the codebase (not the primary one) won't match even if `languagesBreakdown` shows some usage of it.

**Can I fetch a private repository?** No — only public repositories are accessible without authentication, so `repoDetail` and both search modes are limited to public repos.

**Why is `license` missing from some repositories?** Not every repository declares a license file GitHub can detect; in that case the field is left out rather than returned empty.

**How many repositories can one run return?** Up to `maxItems`, capped at 100 per run.

**Is this affiliated with GitHub?** No. This is an independent, third-party actor built on GitHub's public REST API; it is not operated by or affiliated with GitHub, Inc.
