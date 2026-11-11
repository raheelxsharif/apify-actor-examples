# X Community Activity & Member Finder

Collect a public X Community's profile, rules, topic, creator, moderators, facepile and active post authors, together with its top and media posts. No login, cookies, API key or browser is needed: every request reads the same public pages and guest endpoints a logged-out visitor gets.

## What you get

- **Community profile:** name, description, member count, creation date, banner, topic, join policy, NSFW flag and question when present
- **Rules:** rule names plus `rulesDetailed` with each rule's description
- **Creator and moderators:** the creator and the full moderator list X exposes on the public About tab, expanded with public profile details
- **Facepile:** the publicly featured member previews and their image URLs
- **Active members:** authors of the collected posts, added as member rows with role `active` — the cookieless substitute for a full member directory
- **Top and media posts:** merge X's ranked community feed, the media grid and the guest community timelines, de-duplicated by post ID
- **Full post detail:** text, timestamps, language, engagement counts, author snapshot, media, hashtags, mentions, links, quoted posts, polls and cards
- **Filters and sorts:** `postSource` all/top/media, sort top/latest/most liked/most viewed, date range, minimum likes/views/replies/retweets, language and verified authors
- **Honest output:** every row has a `type` (`community`, `post`, `member` or `status`); problems produce a `status` row instead of failing the run, and empty fields are omitted — the dataset never contains `null`, `""`, `[]` or `{}`

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `communityUrls` | array | – | **Required.** Community links or numeric IDs. Accepts `x.com`, `www.x.com` and `twitter.com` links. |
| `postSource` | select | `all` | `all` merges the top and media feeds; `top` collects the ranked feed; `media` collects posts with photos or videos. |
| `sort` | select | `top` | `top` keeps X's popular order; `latest`, `mostLiked` and `mostViewed` re-sort the collected rows. |
| `maxPosts` | integer | `50` | Upper bound per community (1–1000). X's public ceiling is roughly 1–66 posts (typically 20–66), so a larger number is not a guarantee. |
| `includeAbout` | boolean | `true` | Add the community row with profile, rules, topic, creator and moderators. |
| `includeMembers` | boolean | `false` | Add member rows for the creator, moderators and facepile. |
| `includeActiveMembers` | boolean | `false` | Add authors of the collected posts as member rows with role `active`. |
| `maxMembers` | integer | `50` | Cap on member rows per community (1–200) when a member option is on. |
| `dateFrom` / `dateTo` | date | – | Keep posts inside this date range. |
| `minimumLikes` | integer | – | Keep posts with at least this many likes. |
| `minimumViews` | integer | – | Keep posts with at least this many views. |
| `minimumReplies` | integer | – | Keep posts with at least this many replies. |
| `minimumRetweets` | integer | – | Keep posts with at least this many retweets. |
| `language` | select | `any` | Keep posts in one of 40 common languages. |
| `authorVerifiedOnly` | boolean | `false` | Keep posts by verified or blue-verified authors. |
| `proxyConfiguration` | object | – | Optional; the actor works without a proxy and uses a direct connection by default. |

### Example: community overview plus top posts

```json
{
  "communityUrls": ["https://x.com/i/communities/1493446837214187523"],
  "postSource": "all",
  "sort": "top",
  "maxPosts": 50,
  "includeAbout": true
}
```

### Example: media posts and active members

```json
{
  "communityUrls": ["1493446837214187523"],
  "postSource": "media",
  "sort": "mostLiked",
  "includeActiveMembers": true,
  "maxMembers": 25,
  "minimumViews": 500
}
```

### Example: two communities, verified authors, date window

```json
{
  "communityUrls": [
    "https://x.com/i/communities/1493446837214187523",
    "https://twitter.com/i/communities/1539231166569410560"
  ],
  "authorVerifiedOnly": true,
  "dateFrom": "2026-01-01",
  "dateTo": "2026-12-31",
  "maxPosts": 100
}
```

## Output

### Community row (`type: "community"`)

`id`, `url`, `name`, `description`, `memberCount`, `createdAt`, `bannerUrl`, `topic`, `question`, `joinPolicy`, `isNsfw`, `rules`, `rulesDetailed`, `trendingHashtags`, `creator`, `moderators`, `moderatorCount`, `memberFacepile`, `memberPreviewImageUrls`, `collectedPostCount`, `uniquePostCount`, `sourcePostCounts`, `collectedAt`.

### Post row (`type: "post"`)

`id`, `url`, `text`, `createdAt`, `language`, `replyCount`, `retweetCount`, `likeCount`, `quoteCount`, `bookmarkCount`, `viewCount`, `engagementRate`, `isReply`, `isQuote`, `isRetweet`, `isNoteTweet`, `conversationId`, `inReplyToTweetId`, `inReplyToUserId`, `inReplyToUsername`, `hashtags`, `mentions`, `urls`, `cashtags`, `media`, `quotedTweet`, `quotedTweetId`, `poll`, `card`, `possiblySensitive`, `author`, `communityId`, `communityName`, `feed`, `collectedAt`.

`feed` records which surface produced the row (`top` or `media`). `engagementRate` is `(likes + retweets + replies) / followers` as a percentage and appears only when the author's follower count is known.

### Member row (`type: "member"`)

`id`, `username`, `name`, `bio`, `followers`, `following`, `postCount`, `mediaCount`, `profileImageUrl`, `bannerUrl`, `profileUrl`, `accountCreatedAt`, `isBlueVerified`, `verified`, `verifiedType`, `isProtected`, `location`, `website`, `role`, `communityId`, `communityName`, `collectedAt`.

`role` is `creator`, `moderator`, `member` (facepile) or `active` (post author). `accountCreatedAt` is the account's public creation date — X does not expose the date a member joined the community.

### Status row (`type: "status"`)

`status` is one of `noInput`, `invalidLink`, `notFound`, `rateLimited`, `noMatchingPosts`, `membersUnavailable`, `sourceChanged` or `error`; `message` explains it.

## Public limits

- **Roughly 1–66 posts per community (typically 20–66).** X shows only the ranked and media slices to logged-out visitors, with no working cursor. The actor returns the union of every public surface and never invents rows to fill `maxPosts`.
- **No live latest feed.** `/latest` returns zero rows for guests; `sort: "latest"` orders the collected top/media posts newest first.
- **No full member directory.** `/members` redirects to login. The actor returns the creator, the exposed moderator list, the facepile and the authors of collected posts.
- **No community or in-community search.** Those routes require login and return no cookieless data.
- **No cookies, login or API key.** There is no credential input, and every field comes from public data.

## FAQ

**Do I need an X account, cookies or a developer key?**  No. The actor reads public community pages and guest endpoints without any credential input. If a saved task still sends `cookies`, they are ignored with a log message.

**How many posts can I get per community?**  Roughly 1–66 unique posts, typically 20–66, depending on the community. `maxPosts` is an upper bound, not a promise, and filters can reduce the result further.

**Why do I get fewer posts than `maxPosts`?**  X exposes only its ranked and media slices to logged-out visitors and has no working public cursor. The actor returns everything reachable and never fabricates rows.

**What is the difference between top, media and latest?**  `postSource: "top"` collects the ranked feed, `"media"` collects posts with photos or videos and `"all"` merges both. `sort: "latest"` re-orders the collected posts by date because X's live latest feed is login-gated.

**Which members are included?**  The creator, the moderator list shown on the public About tab, the facepile and — when enabled — the authors of the collected posts. A full member directory needs login, so it is not offered.

**Why are dates or counts missing on a few rows?**  Some media-grid rows only expose their date and counts through a second public guest timeline. The actor merges those twins automatically and still omits any value X genuinely does not publish.

**Can I collect a private or NSFW community?**  Public communities work, including NSFW-flagged ones. Private, deleted or login-only communities return a `notFound`, `rateLimited` or `sourceChanged` status row.

**How fresh is the data?**  Live — every run reads X at request time. Schedule the actor for hourly or daily monitoring.

**How do I use the results?**  Download the dataset from the run page, or consume it through the Apify API, webhooks, Zapier, Make, Google Sheets or any other integration.

**Is this affiliated with X?**  No. It is an independent third-party actor that reads publicly available community data.
