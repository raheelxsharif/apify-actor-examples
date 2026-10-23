# YouTube Community Posts, Polls & Comments Scraper

Read a YouTube channel's Posts (Community) tab — or a single post URL — and get clean records for text posts, image carousels, shared videos, polls, quizzes, playlists and reposts. Exact publish dates and like counts, comment counts and comment threads are available without logging in.

## What this actor does

- **Three modes:** `channelPosts` reads a channel feed, `postDetails` enriches one post per URL or ID, and `postComments` returns one row per comment.
- **Every public post type:** text, single image, image carousel, shared video, text poll, image poll, quiz, shared playlist and repost with the original post's content and attachments.
- **Exact engagement:** exact ISO publish date, exact like count and exact comment count when details are enabled; the text YouTube displays is kept too.
- **Comments that work:** up to 1,000 top-level comments per post, Top or Newest order, and optional replies.
- **Filters (in `channelPosts` mode):** post type, keyword and exact publish-date range.
- **Clean output:** one best-resolution URL per image, resolved external links, hashtags and normalized UTC dates. Empty fields are omitted.

## Output per post

- `recordType` — `community_post` for a post, `comment` in postComments mode, or `status` for an explanatory row
- `postId`, `postUrl` — YouTube's post ID and canonical link
- `postType` — `text`, `image`, `video`, `poll`, `quiz`, `playlist` or `shared`
- `channelId`, `channelName`, `channelHandle`, `channelUrl` — the channel whose tab was read
- `channelSubscriberCountText` — subscriber label shown on the channel page
- `authorName` — the post or repost author
- `contentText` — post text as YouTube displays it; a long URL may be shortened here, with the full target in `links[]`
- `links[]` — full external URLs found in the post text
- `hashtags[]` — hashtags found in the post text
- `publishedTimeText`, `isEdited` — relative label and edited marker
- `publishedAt` — exact ISO 8601 UTC publish date (details on)
- `voteCountText`, `likeCount` — displayed likes and exact likes
- `commentCountText`, `commentCount` — displayed and exact comment counts
- `commentsDisabled` — present when the creator turned comments off
- `images[]`, `imageDetails[]` — one best-resolution URL per image, with dimensions
- `videoAttachments[]` — shared video ID, title, URL, thumbnail, description, views, length, owner and badges
- `poll` — `pollChoices`, optional `pollOptions` with images, total votes text and parsed approximate total
- `quiz` — choices with `isCorrect` and `explanation`, plus total answers
- `playlist` — playlist ID, title, video count, URL and thumbnail
- `sharedPost` — a repost's original post: author, text, date label, likes label and attachments
- `isMembership` — present only when YouTube exposes a members-only badge
- `commentPreview[]` — first three comments when details are on but full comments are not requested
- `comments[]` — comment rows with author, text, time, likes, reply count and optional `replies[]`
- `scrapedAt` — ISO 8601 scrape timestamp

## Output per comment (`postComments` mode)

One row per top-level comment and, when enabled, one row per reply:

- `recordType` — `comment`
- `postId`, `postUrl`, `channelId`, `channelName`, `channelHandle`, `channelUrl`
- `commentId`, `parentCommentId`, `replyLevel`
- `authorName`, `authorChannelId`, `authorChannelUrl`, `authorHandle`
- `text`, `publishedTimeText`, `likeCountText`, `replyCount`
- `isCreator`, `isVerified`, `isPinned`, `isHearted`
- `scrapedAt`

Status rows use `recordType: "status"` with a `status` value such as `channel_not_found`, `no_community_tab`, `post_not_found` or `comments_disabled`.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `channelPosts` | `channelPosts`, `postDetails` or `postComments` |
| `urls` | array | – | Channel URLs/handles/IDs, or post URLs/IDs for the post modes |
| `maxPosts` | integer | `100` | Maximum posts per channel in `channelPosts` mode, or per run in the post modes (1–200). |
| `postTypes` | array | `[]` | `channelPosts` only: keep text/image/video/poll/quiz/playlist/shared posts |
| `keyword` | string | – | `channelPosts` only: keep posts whose text contains this phrase |
| `publishedAfter` | string | – | `channelPosts` only: keep posts published on or after this date |
| `publishedBefore` | string | – | `channelPosts` only: keep posts published on or before this date |
| `includeDetails` | boolean | `true` | `channelPosts` only: fetch exact date, likes, comment count and a 3-comment preview. `postDetails` always returns details. |
| `includeComments` | boolean | `false` | Include comment threads in `channelPosts` and `postDetails`; `postComments` always returns comments |
| `maxComments` | integer | `100` | Maximum top-level comments per post (1–1,000) |
| `includeReplies` | boolean | `false` | Include replies under comments |
| `maxRepliesPerComment` | integer | `10` | Maximum replies per comment (1–100) |
| `commentSort` | string | `top` | `top` or `newest` when comments are requested |
| `market` | string | `US` | Country used for YouTube's localized labels |
| `language` | string | `en` | Interface language for relative times and labels |

### Example: recent posts from two channels

```json
{
  "mode": "channelPosts",
  "urls": ["https://www.youtube.com/@MrBeast", "@GoodMythicalMorning"],
  "maxPosts": 20
}
```

### Example: polls and quizzes only

```json
{
  "mode": "channelPosts",
  "urls": ["@veritasium", "@watchmojo"],
  "postTypes": ["poll", "quiz"],
  "maxPosts": 50
}
```

### Example: exact date range with comments

```json
{
  "mode": "channelPosts",
  "urls": ["@MrBeast"],
  "publishedAfter": "2026-08-01",
  "publishedBefore": "2026-10-01",
  "includeComments": true,
  "maxComments": 100,
  "commentSort": "newest"
}
```

### Example: one post by URL, with replies

```json
{
  "mode": "postDetails",
  "urls": ["https://www.youtube.com/post/UgkxZusu9I1Z-VuU5PGZNA2gclHi8V9CJVZk"],
  "includeComments": true,
  "maxComments": 200,
  "includeReplies": true,
  "maxRepliesPerComment": 20
}
```

### Example: comments as one row per comment

```json
{
  "mode": "postComments",
  "urls": ["UgkxZusu9I1Z-VuU5PGZNA2gclHi8V9CJVZk"],
  "maxComments": 500,
  "commentSort": "newest"
}
```

## Use cases

- **Creator agencies** — track announcements, product drops and audience polls across a roster of channels
- **Social listening** — export poll choices, post text and comment sentiment for brand monitoring
- **Competitive research** — compare engagement and posting cadence across creators
- **Content archiving** — save time-limited Community posts with images, links and comments
- **AI datasets** — collect clean creator text, quizzes and threaded comments
- **Community management** — monitor new comments on a post without opening YouTube

## FAQ

**What is a Community post?**
Any text update, image, image carousel, shared video, poll, quiz, playlist or repost a channel publishes to its Posts tab — the feed that sits beside Videos and Shorts.

**Do I need to log in or provide cookies?**
No. The actor reads the same public pages a logged-out visitor sees. Posts that a creator restricted to channel members are not visible without a membership, so they are not returned.

**Why does a channel stop at 200 posts?**
YouTube's Posts tab only serves the newest 200 posts. There is no public way to page further back. If you already have an older post URL or ID, the `postDetails` and `postComments` modes can still read it.

**Why are poll percentages missing?**
YouTube only reveals per-choice votes and percentages to signed-in viewers who voted in that poll. Logged out, the page carries the choices and a rounded total vote count, which the actor returns.

**Are like and comment counts exact?**
Yes when `includeDetails` is on: the actor reads YouTube's exact counters. Counts are a point-in-time snapshot and can change after the scrape. When details are off, only the display text such as `147K` is available. Reposts do not have their own like or comment counters.

**Can I get every comment on a post?**
Up to `maxComments` top-level comments. YouTube's featured-comment feed is finite; for deeper requests the actor continues with the newest-comment feed and removes duplicates. `includeReplies` adds reply threads.

**Why does a post have no comment count?**
The creator turned comments off. The record then carries `commentsDisabled: true` instead of inventing zero.

**Does the market or language field translate post text?**
No. It changes YouTube's own labels and relative times, never the creator's text.

**Can I filter by date, type or keyword?**
Yes, in `channelPosts` mode. `publishedAfter` and `publishedBefore` use the exact publish date, `postTypes` filters attachments, and `keyword` matches the post text case-insensitively. The post modes process the URLs/IDs you give them directly.

**Is this affiliated with YouTube?**
No. This is an independent third-party actor that reads public YouTube pages and is not operated by or affiliated with YouTube or Google.
