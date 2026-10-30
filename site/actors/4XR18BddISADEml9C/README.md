# Snapchat Spotlight Comment & Reaction Scraper

Snapchat embeds a set of top comments directly in the public page for each Spotlight video. This actor reads that data for one or more specific videos, or for every pinned Spotlight video on a creator's profile, and returns commenter names, Bitmoji IDs, reaction counts, posting timestamps, and reply counts — with full Unicode support for scripts like Urdu and Arabic.

## What this actor does

- Fetches comments for one or more specific Spotlight video URLs (`bySpotlightUrl`).
- Fetches comments for every pinned Spotlight video on a creator's profile in a single request per creator (`byUsername`).
- Returns commenter display name, Bitmoji avatar/selfie IDs, and profile image for each comment.
- Captures per-comment reaction counts, reply counts, and moderation status.
- Emits an error record (rather than failing the run) when a video or username has no reachable comments.

## Output per comment

- `commentId` — identifier composed of `{highBits}-{lowBits}`
- `snapId` — the Spotlight video's snap ID
- `sourceUsername` — creator username (`byUsername` mode)
- `inputUrl` — the Spotlight URL that was scraped (`bySpotlightUrl` mode)
- `replyText` — comment text (full Unicode, including Arabic, Urdu, and emoji)
- `reactionCounts` — array of `{reactTypeId, count}` objects, one per reaction type used on the comment
- `commenterDisplayName`, `commenterBitmojiAvatarId`, `commenterBitmojiSelfieId`, `commenterProfileLogoUrl`
- `reportCount` — number of times the comment has been reported
- `approvalState` — moderation state code (`3` = approved and visible)
- `postedAt` — ISO 8601 UTC timestamp the comment was posted
- `rankingScore` — Snapchat's own ranking score for the comment, when present
- `threadedReplyCount` — number of threaded replies to the comment
- `scrapedAt` — ISO 8601 UTC scrape timestamp

A failed lookup produces an error record instead, with `inputUrl` or `inputUsername`, `snapId` (when it was resolved before the failure), `error`, and `scrapedAt`.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `byUsername` | `bySpotlightUrl` (by direct video URL) or `byUsername` (from a creator's profile page) |
| `spotlightUrls` | array | – | Spotlight video URLs, e.g. `https://www.snapchat.com/spotlight/{id}`; used by `bySpotlightUrl` |
| `usernames` | array | – | Snapchat creator usernames; used by `byUsername` |
| `maxCommentsPerSnap` | integer | `100` | Maximum comments to extract per Spotlight video (1-1000) |
| `proxyConfiguration` | object | – | Optional Apify Proxy configuration for the run |

### Example: comments from two creators' pinned videos

```json
{
  "mode": "byUsername",
  "usernames": ["waqarzaka", "brentrivera"],
  "maxCommentsPerSnap": 100
}
```

### Example: comments from a specific video

```json
{
  "mode": "bySpotlightUrl",
  "spotlightUrls": [
    "https://www.snapchat.com/spotlight/W7_EDlXWTBiXAEEniNoMPwAAYaXFxdmVobmp6AZ499vl-AZ499rsXAAAAAQ"
  ],
  "maxCommentsPerSnap": 200
}
```

## Use cases

- Audience sentiment research on a creator's Spotlight content.
- Track reaction and reply counts on specific videos over time.
- Build a moderation or spam-review dataset from public comment text.
- Compare comment activity across multiple creators in one run.

## FAQ

**Why are some Spotlight videos missing comments?**
Snapchat only shows comments on its public page for videos with active, visible comments. Videos with no comments, disabled comments, or very recent posts may return nothing; the actor records an error row for those instead of failing the run.

**What's the difference between the two modes?**
`byUsername` reads a creator's public profile page, which carries comments for all their pinned Spotlight videos in one request — efficient for getting everything from one creator. `bySpotlightUrl` reads each individual video page separately, which works for specific videos that aren't pinned on a profile.

**Are these all comments on a video, or just the top ones?**
Snapchat's own public page shows a curated set of top-ranked comments, not the full comment history. How many are shown varies by video; `maxCommentsPerSnap` caps how many this actor extracts per video.

**Do I need a Snapchat account, login, or cookies?**
No. This data is visible on Snapchat's public pages to any visitor, without an account.

**Can I scrape comments from multiple creators at once?**
Yes — list multiple usernames; each creator's comments are tagged with their own `sourceUsername`.

**What does `approvalState` mean?**
It's Snapchat's own moderation state for the comment. `3` means approved and visible; other values indicate the comment is pending review or was otherwise moderated.

**Does this support non-English comments?**
Yes. Full Unicode is supported, including Arabic, Urdu, emoji, and other non-ASCII character sets.

**Is this affiliated with Snapchat?**
No. This actor reads data shown on Snapchat's own public pages and has no relationship with Snap Inc.
