# LinkedIn Profile Engagement Timeline Scraper

Pull a LinkedIn profile's engagement history in one run: the posts and reposts they published, the comments they left on other people's posts, and the reactions (likes) they gave — each returned as its own record with the original post's author, text, and link.

## What this actor does

- Fetches posts/reposts, comments, and reactions for one or more LinkedIn profiles in a single run, using LinkedIn's own activity feed modules.
- Lets you pick which activity categories to pull via `activityTypes`, or scrape all three at once.
- Narrows results to an ISO date window with `minDate`/`maxDate` so you only pull recent engagement.
- Attaches the original post's author, headline, avatar, and post URL to every like, comment, and repost, plus media (images, video, article) for the profile's own posts.
- Fields that don't apply to a given activity (for example `commentText` on a like, or `mediaItems` on a text-only post) are left out of that record.

## Authentication

This actor requires a LinkedIn session cookie (`cookie`) because activity history is only served to a logged-in session. Supply either the `li_at` cookie value from your browser's DevTools, or a full cookie-export JSON array. If the cookie is missing or has expired, the actor returns a single status record describing the problem instead of failing silently.

## Output per activity

- `activityType` — `post`, `repost`, `comment`, or `reaction`
- `originalPostUrl` — URL of the post the activity relates to
- `originalPostId` — LinkedIn activity ID of that post
- `originalPostedAt` — when the original post was published
- `originalAuthorName`, `originalAuthorHeadline`, `originalAuthorAvatar` — the post's author
- `originalAuthorProfileUrl` — profile URL of that author
- `content` — text of the original post
- `reactedAt` — when the profile performed this activity (liked, commented, posted)
- `targetAuthor` — the person whose post was reacted to or commented on, for reactions/comments/reposts
- `targetPostUrl` — present when the activity references a distinct target post URL
- `reactionType` — the reaction given (e.g. `LIKE`, `PRAISE`), for `reaction` activities
- `commentText` — the text the profile actually wrote, for `comment` activities
- `mediaItems` — images, video, or article attachments, for the profile's own posts
- `inputProfileUrl` — the profile URL you supplied
- `scrapedAt` — ISO 8601 timestamp of when the record was scraped

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `profileUrls` | array | – | LinkedIn profile URLs or bare usernames (required) |
| `cookie` | string | – | LinkedIn session cookie — `li_at` value or full cookies JSON array (required) |
| `activityType` | string | `all` | Legacy single-select: `all`, `likes`, `comments`, or `posts`. Kept for backwards compatibility — prefer `activityTypes` |
| `activityTypes` | array | `["posts","comments","reactions","shares"]` | Multi-select of `posts`, `comments`, `reactions`, `shares` to scrape |
| `minDate` | string | – | Only include activity on or after this ISO date |
| `maxDate` | string | – | Only include activity on or before this ISO date |
| `language` | string | – (auto) | Preferred response language for LinkedIn's API: `en`, `es`, `fr`, `de`, or `pt` |
| `maxActivitiesPerProfile` | integer | `50` | Max activity items per profile, per activity type (1–500) |
| `proxyConfiguration` | object | – | Apify proxy settings; a residential proxy is recommended for reliability |

### Example: full engagement history for two profiles

```json
{
  "profileUrls": ["https://www.linkedin.com/in/williamhgates/", "adam-grant"],
  "cookie": "AQEDATVc5uMEM5s_AAAB...",
  "activityTypes": ["posts", "comments", "reactions", "shares"],
  "maxActivitiesPerProfile": 50
}
```

### Example: only comments, within a date window

```json
{
  "profileUrls": ["https://www.linkedin.com/in/satyanadella"],
  "cookie": "AQEDATVc5uMEM5s_AAAB...",
  "activityTypes": ["comments"],
  "minDate": "2026-01-01",
  "maxDate": "2026-06-30"
}
```

### Example: just posts and reposts, higher cap

```json
{
  "profileUrls": ["https://www.linkedin.com/in/adam-grant/"],
  "cookie": "AQEDATVc5uMEM5s_AAAB...",
  "activityTypes": ["posts"],
  "maxActivitiesPerProfile": 200
}
```

## Use cases

- **Sales intelligence** — see which posts and topics a prospect engages with before an outreach call
- **Influencer analysis** — map which content a thought leader likes, comments on, and shares
- **Competitive research** — track what competitors' employees react to or publish
- **Lead enrichment** — add recent LinkedIn engagement signals to CRM contacts
- **Content strategy** — study which post types draw reactions and comments from a target audience

## FAQ

**Why does this need a cookie instead of just a profile URL?**
LinkedIn only serves activity history (likes, comments, post history) to an authenticated session — there's no public, logged-out version of this data, unlike a profile's headline or job history.

**What's the difference between `activityType` and `activityTypes`?**
`activityTypes` is the current multi-select input and should be used going forward; `activityType` is a legacy single-select kept only for older integrations.

**Why do I get a status record instead of activity data?**
If the cookie is missing, malformed, or has expired, the actor returns one record describing the problem instead of failing the whole run, so you can see exactly what to fix.

**Can I see activity from a profile I'm not connected to?**
Only if your cookie's account can already view that profile's activity on linkedin.com. The actor can't surface anything your session wouldn't see on the site.

**Why is `targetPostUrl` sometimes missing on a comment or reaction?**
It's only included when LinkedIn's response distinguishes a separate target post URL from `originalPostUrl`; when the two are the same, the field is omitted.

**How far back can I pull activity?**
As far as LinkedIn's own activity feed modules return for that profile — older activity may not be available if LinkedIn itself has stopped surfacing it.

**Is this affiliated with LinkedIn?**
No. This is an independent, third-party actor that uses your own session cookie to read activity data LinkedIn already shows that account; it is not operated by or affiliated with LinkedIn.
