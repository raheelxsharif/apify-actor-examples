# YouTube Community Tab Post & Poll Scraper

Read a channel's Community tab the way a subscriber would — text updates, image posts, polls, and shared videos — and get each post back as a structured record, with its vote and comment counts, instead of scrolling the feed yourself.

## What this actor does

- Scrapes public Community-tab posts from one or more YouTube channels, accepting channel URLs, @handles, or channel IDs.
- Captures post text, all attached images (including multi-image galleries), shared video attachments, and poll choices with vote totals.
- Can optionally pull the comment thread exposed under each post, up to a configurable cap.
- Lets you choose a market/country code so posts render the way they would for a viewer in that locale.
- When a channel has no public Community posts, the actor returns a single status record explaining that instead of an empty or fabricated result; otherwise empty fields are left out of each post record.

## Output per post

- `recordType` — `community_post` for a normal post, or `status` for a channel-level status row
- `postId` — YouTube's identifier for the post
- `postUrl` — link to the post on youtube.com
- `channelId`, `channelHandle` — the channel the post belongs to
- `authorName` — the post's author, typically the channel itself
- `contentText` — the post's text
- `publishedTimeText` — relative publish time as shown on YouTube (e.g. `"2 days ago"`)
- `voteCountText` — like/vote count as displayed
- `commentCountText` — comment count as displayed
- `images[]` — URLs of attached images, including multi-image gallery posts
- `videoAttachments[]` — shared videos, each with `videoId`, `title`, `url`, `viewCountText`, `publishedTimeText`, `lengthText`
- `poll` — `pollChoices[]`, `pollTotalVotesText`, `pollType`, present only on poll posts
- `comments[]` — comment threads (`commentId`, `authorName`, `authorChannelId`, `text`, `publishedTimeText`, `likeCountText`), present only when `includeComments` is enabled
- `scrapedAt` — ISO 8601 timestamp of when the record was scraped
- A `status` row (when a channel has no public posts, or its page couldn't be fetched) carries `status`, `channelUrl`, `channelId`, `channelHandle`, and sometimes `error` instead of post fields.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `channelUrls` | array | – | Channel URLs, @handles, or channel IDs (required) |
| `maxPosts` | integer | `100` | Maximum post rows to return per channel (1–10000) |
| `includeComments` | boolean | `false` | Include comments exposed on the post page |
| `maxComments` | integer | `100` | Maximum comments to keep per post when enabled (1–1000) |
| `market` | string | `US` | Country code used to render the public, localized channel page |

### Example: posts from two channels, no comments

```json
{
  "channelUrls": ["https://www.youtube.com/@MrBeast", "https://www.youtube.com/@RickAstleyYT"],
  "maxPosts": 20
}
```

### Example: posts with their comment threads

```json
{
  "channelUrls": ["@MrBeast"],
  "maxPosts": 10,
  "includeComments": true,
  "maxComments": 20
}
```

### Example: large pull from one channel in a specific market

```json
{
  "channelUrls": ["@BLACKPINK"],
  "maxPosts": 500,
  "market": "KR"
}
```

## Use cases

- **Community management** — track what a channel is posting to its Community tab without checking manually
- **Poll tracking** — collect poll questions, choices, and vote totals from creators you follow
- **Engagement research** — compare vote and comment counts across a channel's post history
- **Content archiving** — preserve time-limited Community posts that can disappear from the feed
- **Competitive monitoring** — watch what competitor channels announce to their audience

## FAQ

**What counts as a "Community post"?**
Any text update, image, image gallery, poll, or shared video a channel publishes to its Community tab — a feed separate from regular video uploads.

**Why did a channel return a `status` record instead of posts?**
Not every channel has posted to its Community tab, or the feature may not be enabled for that channel; the status record explains why no posts were returned instead of leaving you with an empty, unexplained result.

**Do I need to log in or provide cookies?**
No — the actor reads the same public Community tab pages a logged-out visitor sees.

**Why are some posts missing `images`, `videoAttachments`, or `poll`?**
A post only has one of these attachment types (or none at all, for plain text); fields that don't apply to a given post are left out rather than returned empty.

**Can I get every comment on a post?**
`includeComments` returns the comments YouTube exposes directly on the Community post page, capped by `maxComments` — it is not a full comment-thread crawl with replies expanded.

**Does the `market` field translate the post text?**
No — it only affects how YouTube's own localized elements (like relative time text) render; the post's own text stays as the creator wrote it.

**Is this affiliated with YouTube?**
No. This is an independent, third-party actor that reads YouTube's public Community tab pages and is not operated by or affiliated with YouTube or Google.
