# Hacker News Archive: Stories, Comments & Hiring

Hacker News is a living archive of technology discussion: front-page links, Ask HN and Show HN threads, job ads, polls, user karma, and years of comments. This actor collects all of it from the public Hacker News surfaces — live feeds, keyword search, historical front pages, individual items, user profiles, and the monthly “Who is hiring?” threads. No login, cookies, API key, or proxy setup is needed.

## What it does

- **11 modes** — six live feeds, item lookup, user profiles, keyword search, historical front pages, and monthly hiring threads.
- **Complete comment trees** — flat rows or nested `comments`, with no duplicated replies and a correct `maxItems` cap.
- **Deep search** — relevance or newest-first ordering, tags, author, story, date range, points, and comment filters.
- **Search past 1,000 results** — adaptive date-window splitting keeps collecting when a query is larger than the search API’s per-query ceiling.
- **Typed records** — stories, comments, jobs, polls with vote options, poll options, user profiles, and parsed job postings.
- **Historical front pages** — read any day back to 2007 with its original ranking, score, author, and comment count.

## Services

The actor reads three public Hacker News services:

| Service | What it provides |
|---|---|
| Firebase Hacker News API | Live feeds, individual items, user profiles, scores, comment counts, and exact deleted/dead flags |
| Algolia Hacker News Search | Keyword search, date ordering, tag and author filters, and fast comment-tree reads |
| Hacker News HTML | Historical `/front?day=` pages and their original story rankings |

All three are reachable without credentials.

## How it works

Pick a mode and the actor chooses the right source for it:

- **Feeds** read the complete live list for the chosen section, then fetch item details and apply your filters. It keeps walking until the feed ends or `maxItems` is reached, so filtered-out stories do not silently shorten the result.
- **Search** sends your query, tags, author, story, and date range to Hacker News search. With `exhaustiveSearch` enabled it splits the date range whenever a window would exceed 1,000 hits, then merges the windows without duplicates.
- **Comments** use the fast threaded read first. When a thread is missing live replies, the actor checks the thread’s comment search and merges the gap. Asking for dead or deleted content switches to the exact live item walk.
- **Historical front pages** read page after page of the chosen day, then deduplicate stories that appear twice.
- **Who is hiring?** finds the requested number of monthly threads, reads their top-level posts, and extracts the common hiring fields without inventing missing details.

Every mode is filter-aware: a story is checked before its comments are fetched, so filtered-out stories cost nothing extra.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `topStories` | `topStories` / `newStories` / `bestStories` / `askStories` / `showStories` / `jobStories` / `item` / `user` / `search` / `frontPage` / `whoIsHiring` |
| `itemIds` | array | `[]` | Item IDs for `mode=item` |
| `usernames` | array | `[]` | Usernames for `mode=user` |
| `searchQueries` | array | `[]` | Keywords for `mode=search`; several queries are merged |
| `searchSort` | string | `relevance` | `relevance` or `date` |
| `contentTypes` | array | `["story"]` | `story`, `comment`, `poll`, `pollopt`, `job`, `show_hn`, `ask_hn`, `front_page` |
| `author` | string | – | Restrict search to one username |
| `storyId` | string | – | Restrict search to one story |
| `exhaustiveSearch` | boolean | `false` | Date-slice past the 1,000-hit search ceiling |
| `dateRangeFrom` | string | – | Earliest date (`YYYY-MM-DD`, UTC) |
| `dateRangeTo` | string | – | Latest date (`YYYY-MM-DD`, UTC) |
| `minScore` | integer | – | Minimum points for stories, jobs, and polls |
| `minCommentCount` | integer | – | Minimum comment count for stories and polls |
| `domainAllowlist` | array | `[]` | Keep only these link-host fragments |
| `domainBlocklist` | array | `[]` | Drop these link-host fragments |
| `excludeDeadOrDeleted` | boolean | `true` | Drop dead/deleted items |
| `commentAuthorFilter` | array | `[]` | Keep only comments by these authors |
| `commentSort` | string | `rank` | `rank` or `newest` sibling order |
| `maxComments` | integer | `0` | Comments collected per story; 0 skips comments |
| `maxDepth` | integer | `5` | Maximum reply depth; roots are depth 0 |
| `enableCommentHierarchy` | boolean | `false` | Nested `comments` instead of flat comment rows |
| `includeSubmissions` | boolean | `false` | Also emit a user’s submissions |
| `submissionTypes` | string | `all` | `all`, `story`, or `comment` |
| `frontPageDate` | string | – | Day for `mode=frontPage`; empty means yesterday |
| `hiringMonths` | integer | `1` | Recent monthly hiring threads to parse |
| `hiringThreadIds` | array | `[]` | Specific hiring threads, overriding the month count |
| `hiringKeywords` | array | `[]` | Keep matching hiring posts |
| `remoteOnly` | boolean | `false` | Keep hiring posts that mention remote work |
| `maxItems` | integer | `50` | Total records emitted; nested comments count too |
| `includeAuthorProfiles` | boolean | `false` | Add author karma and account creation date |
| `startUrls` | array | `[]` | Item, user, front-page, or hiring-thread links |

Empty fields are omitted from the output — there are no nulls or blank strings.

### Example: search with filters

```json
{
  "mode": "search",
  "searchQueries": ["rust async runtime", "tokio"],
  "searchSort": "date",
  "contentTypes": ["story", "comment"],
  "dateRangeFrom": "2025-01-01",
  "minScore": 50,
  "maxItems": 200
}
```

### Example: full comment tree

```json
{
  "mode": "item",
  "itemIds": ["49922569"],
  "maxComments": 300,
  "maxDepth": 10,
  "enableCommentHierarchy": true,
  "commentSort": "newest"
}
```

### Example: a historical front page

```json
{
  "mode": "frontPage",
  "frontPageDate": "2024-01-01",
  "maxItems": 500
}
```

### Example: this month’s hiring posts

```json
{
  "mode": "whoIsHiring",
  "hiringMonths": 1,
  "hiringKeywords": ["python", "backend"],
  "remoteOnly": true,
  "maxItems": 100
}
```

## Output per story, job, or poll

- `id`, `hnUrl`, `type` (`story`, `job`, or `poll`), `title`
- `url`, `domain` for link posts
- `score`, `author`, `numComments`
- `createdAt` (ISO-8601 UTC), `createdAtEpoch`, `ageHours`
- `text` for Ask HN, Show HN, jobs, and polls
- `kids` — top-level comment IDs
- `rank` — feed, search, or front-page position
- `frontPageDate` for historical rows
- `pollOptions[]` — `id`, `text`, `votes`, `author`, `createdAt`, `hnUrl`
- `fetchedCommentCount` when comments were read
- `comments[]` — nested replies when hierarchy is enabled
- `authorKarma`, `authorCreatedAt` when author enrichment is enabled
- `dead` / `deleted` only when true
- `recordType: "story"`, `scrapedAt`

## Output per comment

- `id`, `hnUrl`, `type`, `author`, `text`
- `storyId`, `parentId`, `depth`
- `createdAt`, `createdAtEpoch`, `ageHours`
- `kids` — reply IDs
- `storyTitle`, `storyUrl` when a search hit supplies them
- `replies[]` in nested mode only
- `dead` / `deleted` only when true
- `recordType: "comment"`, `scrapedAt`

## Output per user

- `id`, `profileUrl`
- `karma`, `createdAt`, `createdAtEpoch`, `about`
- `submittedCount`, `submitted` (first 50 recent IDs)
- `recordType: "user"`, `scrapedAt`

With `includeSubmissions`, normal story and comment rows follow the profile.

## Output per poll option

- `id`, `hnUrl`, `type: "pollopt"`, `text`, `votes`, `pollId`
- `author`, `createdAt`, `createdAtEpoch`, `ageHours`
- `recordType: "pollOption"`, `scrapedAt`

## Output per hiring post

- `id`, `hnUrl`, `threadId`, `threadTitle`, `threadMonth`
- `author`, `company`, `role`, `location`, `remote`, `salary`
- `techStack`, `visa`, `email`, `applyUrl`
- `text`, `postedAt`
- `recordType: "jobPosting"`, `scrapedAt`

Parsed hiring fields are omitted when the post does not state them.

## Usage cost

There is no paid data source, proxy, or credential in this actor. Usage is ordinary Apify compute plus the records you choose to collect. The cheapest useful run is the default prefill: 20 top stories with comments and author enrichment off. Cost grows with `maxItems`, `maxComments`, `maxDepth`, multiple search queries, `exhaustiveSearch`, and `includeAuthorProfiles` because each adds fetches. Small `maxItems` values keep runs short and inexpensive.

## FAQ

**Do I need an account, cookie, or API key?** No. Every mode reads public Hacker News data.

**Do I need a proxy?** No. The actor connects directly and needs no proxy configuration.

**Why is a field missing from some records?** Hacker News does not store that field for every item. For example, self posts have text but no link URL, and many users have no “about” text. Missing fields are omitted instead of filled with a placeholder.

**Why do comment scores never appear?** Hacker News does not publish per-comment scores through its feeds, search, or HTML. Stories, polls, jobs, and poll options do have scores.

**Why did a search stop at 1,000 results?** Hacker News search returns at most 1,000 hits per query window. Turn on `exhaustiveSearch` to split the date range into smaller windows and collect deeper results.

**Why is relevance order different when `exhaustiveSearch` is on?** The deep search collects several date windows. Each window is ranked by relevance, so the final list follows window order rather than one global relevance ordering. Use `searchSort: "date"` for a stable chronological order.

**How fresh is the data?** Feeds and item lookups are live. Search and comment search can lag the live site by a short period, so brand-new comments or score changes may appear slightly later.

**What is the difference between `maxItems` and `maxComments`?** `maxItems` caps every emitted record, including nested comments. `maxComments` caps how many comments are read for each story before depth filtering.

**Can I read a historical best, Ask HN, or Show HN list?** No. Hacker News archives only the historical front page by date. Live Ask HN, Show HN, best, and job lists are available through their normal modes.

**Is this affiliated with Hacker News or Y Combinator?** No. This is an independent actor that reads Hacker News’ public data.

## Notes

- All timestamps are UTC and ISO-8601, with a matching Unix epoch.
- Records are snapshots: scores, comment counts, and comment trees can change after a run.
- Flat comment output contains one row per comment and never repeats a reply inside another row.
- Nested comment output is one story row; every reply carries its own type and scrape timestamp.
- `item` returns a comment row for a comment ID, a story-style row for a story, job, or poll, and a poll-option row for a poll-option ID.
- `startUrls` accepts item, user, historical front-page, and hiring-thread links alongside the mode-specific lists.
- Historical front pages are deduplicated when a story appears twice on the same day.

## Limitations

- **No comment scores or flags.** Hacker News does not expose per-comment scores, moderation flags, or edit history publicly.
- **Search ceiling.** Each Hacker News search window returns at most 1,000 hits. `exhaustiveSearch` works around this by date slicing, but a single second with more than 1,000 hits cannot be split further.
- **Historical lists.** Only the front page is archived by day. Historical best, Ask HN, Show HN, and jobs pages are not available.
- **Feed caps.** Hacker News itself limits the live feeds: top and new stories usually expose 500 IDs, best about 200, Ask HN 20, Show HN up to about 150, and jobs about 30. The actor reports when a feed is exhausted.
- **Comment-tree lag.** The fast tree can trail the live thread by a short period, and deleted branches may be absent. The actor merges a comment-search fallback when live replies are missing; turning off `excludeDeadOrDeleted` uses the exact slower walk.
- **Hiring parsing is heuristic.** Company, role, location, salary, tech stack, visa, email, and apply link are extracted from free-form posts. Fields the post does not state are omitted; no value is invented.
- **Very large user histories.** A user can have tens of thousands of submissions. `maxItems` and `submissionTypes` keep the run bounded.
- **Relevance is not deterministic.** Search relevance can shift between runs as the index changes; date order is stable.
