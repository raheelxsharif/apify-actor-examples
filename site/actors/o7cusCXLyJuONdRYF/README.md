# X Article Announcement Post Metadata Collector

X Articles are long-form posts that live behind a card on a regular post — the "announcement post." This actor takes that announcement post (or the article's own link) and returns the article's **full text** — as plain text, Markdown, and rich content blocks — plus its title, preview, cover image, dates, inline media, resolved links, and the announcement post with its author and engagement metrics. No login required. Article reading on x.com itself is behind a sign-in wall; this actor reads the public article payload X still serves to logged-out pages.

## What this actor does

- **Full article body:** `bodyText`, `bodyMarkdown`, the raw `contentBlocks` (with headings, lists, quotes, links and inline images), resolved `links` and inline `media`
- **Article lookup by announcement post or article link:** paste the post that opens the article, or an article link
- **Public metadata:** `articleId`, `articleUrl`, `title`, `previewText`, `coverImageUrl` (with dimensions), `publishedAt`, `modifiedAt`
- **Announcement post:** optional full post row (text, author, metrics, media) attached to each article
- **Author and engagement:** the author's public profile with `authorFollowers`, plus a derived `engagementRate` and post-level `likeCount` / `replyCount` / `retweetCount` / `quoteCount` / `bookmarkCount` / `viewCount`
- **Honest provenance:** `bodyAvailable` and `bodySource` (`x` or `fxtwitter`) tell you exactly whether and where the body came from
- **Status rows:** invalid links, deleted posts, posts without an article, raw article links that X cannot map, and body-less articles all produce an explicit `status` row with a message
- **Empty fields are omitted** — the dataset never contains `null`, `""`, `[]` or `{}`

## Output per article

- `articleId` — X's article ID
- `articleUrl` — canonical article link (opens for signed-in readers)
- `title` — article title
- `previewText` — the short preview text X shows for the article (omitted when **Include preview text** is off)
- `coverImageUrl`, `coverImageWidth`, `coverImageHeight` — cover image and dimensions (omitted when **Include cover image** is off)
- `bodyAvailable` — `true` when the full body was collected
- `bodyText` — complete article text
- `bodyMarkdown` — the same body converted to Markdown (headings, lists, quotes, links, images)
- `contentBlocks` — the raw rich-text blocks (`type`, `text`, ranges) for custom rendering
- `links[]` — `{text, url}` for every link inside the article
- `media[]` — inline images `{url, width, height, mediaId, position}`
- `bodySource` — `x` when the body came from X's own article payload, `fxtwitter` when the public mirror was used
- `publishedAt`, `modifiedAt` — publication and last-modification timestamps
- `sourceTweetId`, `sourceTweetUrl` — the announcement post
- `author`, `authorFollowers` — the author's public profile and follower count
- `language` — post language code
- `announcementTweet` — the full announcement post when **Include announcement post** is on
- `likeCount`, `replyCount`, `retweetCount`, `quoteCount`, `bookmarkCount`, `viewCount` — announcement post metrics
- `engagementRate` — derived `(likes + retweets + replies) / followers` as a percentage; present only when the follower count is known
- `type: "article"`, `collectedAt`

### Sample record

```json
{
  "type": "article",
  "articleId": "1979875955100258304",
  "articleUrl": "https://x.com/i/article/1979875955100258304",
  "title": "Wtf are these Threat Actors doing? XUbuntu malware is dumb and stinky",
  "previewText": "Yesterday night before I went to bed I saw @sysadafterdark write they suspected...",
  "coverImageUrl": "https://pbs.twimg.com/media/G3n2yXwW8AALy8G.jpg",
  "bodyAvailable": true,
  "bodyText": "Yesterday night before I went to bed I saw @sysadafterdark write they suspected Xubuntu-dot-org was compromised...",
  "bodyMarkdown": "Yesterday night before I went to bed I saw [@sysadafterdark](https://x.com/sysadafterdark) write...",
  "bodySource": "x",
  "publishedAt": "2025-10-19T12:21:52+00:00",
  "sourceTweetId": "1979885711579865307",
  "sourceTweetUrl": "https://x.com/i/status/1979885711579865307",
  "author": { "username": "vxunderground", "name": "vx-underground", "followers": 454667 },
  "authorFollowers": 454667,
  "likeCount": 554,
  "engagementRate": 0.1381,
  "collectedAt": "2026-10-17T00:00:00+00:00"
}
```

## Output per status row

A `status` row is emitted when an input cannot be collected (or when a body could not be attached). It always has a `message`:

- `type: "status"`
- `status` — `noInput`, `invalidLink`, `notFound`, `articleLinkUnmapped`, `noArticle`, `bodyUnavailable`, `rateLimited` or `error`
- `tweetUrl`, `tweetId` — the input it applies to
- `collectedAt`

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `tweetUrls` | array | – | **Required.** Announcement post links (`x.com/<user>/status/<id>`), post IDs, or article links (`x.com/i/article/<id>`). |
| `includeArticleBody` | boolean | `true` | Include the full article text, Markdown, rich blocks, links and inline media. |
| `includeAnnouncementPost` | boolean | `true` | Attach the full announcement post (text, author, metrics, media) to every article row. |
| `includeAuthorProfile` | boolean | `true` | Enrich the author with their public profile (follower count, bio, verification) so `engagementRate` can be computed. |
| `includeCoverImage` | boolean | `true` | Include the article's cover image link and dimensions. |
| `includePreviewText` | boolean | `true` | Include X's short preview text for the article. |
| `maxItems` | integer | `1000` | Stop after this many article rows (1–1000). Status rows are not counted. |

### Example: one announcement post

```json
{
  "tweetUrls": ["https://x.com/vxunderground/status/1979885711579865307"]
}
```

### Example: several articles, metadata and body

```json
{
  "tweetUrls": [
    "https://x.com/vxunderground/status/1979885711579865307",
    "https://x.com/thedankoe/status/2010751592346030461"
  ],
  "maxItems": 10
}
```

### Example: metadata only, no body

```json
{
  "tweetUrls": ["https://x.com/satyanadella/status/2066182223213293753"],
  "includeArticleBody": false,
  "includeAnnouncementPost": false
}
```

### Example: article link form

```json
{
  "tweetUrls": ["https://x.com/i/article/1979885711579865307"],
  "includeAnnouncementPost": true
}
```

## Use cases

- **LLM and RAG pipelines** — feed complete X Articles as Markdown into vector stores, NotebookLM, or chat assistants
- **Content intelligence** — track which articles a set of accounts is publishing, with titles, bodies and cover images
- **Media monitoring** — collect article announcements alongside their engagement metrics
- **Research and archiving** — keep a structured, full-text index of public X Articles and their authors
- **Competitive analysis** — compare article output and performance across creators or brands
- **Newsroom workflows** — enrich a list of announcement posts with full article text and metadata

## Limitations

- **Raw article links cannot be resolved on their own.** X maps an article to its announcement post only for signed-in readers; for everyone else the article page is a sign-in wall. When you paste `x.com/i/article/<id>`, the actor returns an `articleLinkUnmapped` status row and tells you to paste the announcement post instead.
- **An account's full article list is not available without signing in.** X's article tab is behind a login and article pages are not indexed by search engines, so paste the announcement links you need (or schedule the actor on the ones you care about).
- **The body fallback is a public mirror.** X occasionally withholds an article payload for older posts; the actor then reads the same article from the public FxTwitter mirror and marks the row with `bodySource: "fxtwitter"` so you always know where the text came from. If every source fails, the row keeps its metadata and a `bodyUnavailable` status row is added.
- **Posts that X removed or editors replaced** can return their newest state only; deleted articles cannot be recovered cookielessly.

## FAQ

**Does this actor return the full article text, or just the preview?**  The full text. X still serves the complete article payload to logged-out clients; the actor collects it as `bodyText`, `bodyMarkdown` and `contentBlocks`, and says `bodyAvailable: true`. The Markdown keeps headings, lists, quotes, links and images, and also renders embedded posts, code blocks and math. If X ever withholds a body, the actor says so via `bodyUnavailable` instead of guessing.

**What is `bodySource`?**  Where the body came from. `x` means X's own public article payload; `fxtwitter` means the public FxTwitter mirror was used as a fallback because X's payload was unavailable at that moment.

**Which links can I paste?**  The announcement post — `https://x.com/<user>/status/<id>` — is the reliable input. Raw `https://x.com/i/article/<id>` links cannot be resolved cookielessly (X maps article IDs to posts only for signed-in readers), so when they cannot be resolved the actor returns an `articleLinkUnmapped` status row explaining what to paste instead.

**How do I find the announcement post for an article?**  Open the article and use the post whose card opens it; the link is usually shared alongside the article.

**What happens with posts that are not articles?**  You get a `noArticle` status row. Deleted or private posts return a `notFound` status row.

**Can I get every article an account published?**  Not cookielessly — X shows an account's article list only to signed-in readers, and article pages are not indexable. Paste the announcement links you need; the actor collects each one.

**Do I need an X account, login or developer key?**  No. The actor reads only public article and post data, and there is no credential input at all.

**How fresh is the data?**  Live — every run reads X at request time. Schedule the actor for daily or weekly refreshes.

**How do I integrate the results?**  Use the dataset from the run page, the Apify API, webhooks, or a no-code connection such as Zapier, Make or Google Sheets.

**Is this affiliated with X?**  No. This is a third-party actor that reads publicly available article and post data and is not affiliated with or endorsed by X Corp.
