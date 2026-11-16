# Patreon Creator Economics & Membership Tier Scraper

Patreon creator pages publish patron counts, membership tier pricing and recent public posts for anyone to see, even without a Patreon account. This actor reads that data for one or more creators, searches across creators by keyword, browses Patreon's Explore directory by category, or fetches a specific public post directly by URL.

## What this actor does

- Fetches a creator's campaign metadata, membership tiers with pricing, and patron/paid-member counts.
- Optionally pulls that creator's recent public posts alongside the campaign record.
- Searches across all of Patreon by keyword, or browses the Explore directory by category.
- Fetches a single public post's detail directly from its URL.
- Filters out creators below a minimum patron count, or marked NSFW, before they're emitted.
- Fields a creator or post doesn't publish (for example a hidden patron count) are left out of that record.

## Output per creator

| Field | Description |
| --- | --- |
| `id` | Patreon campaign ID |
| `vanity` | Vanity slug (`PhilosophyTube`) |
| `url` | Public Patreon URL |
| `name` | Campaign name |
| `summary` | Plain-text campaign description |
| `oneLiner` | Short tagline |
| `creationName` | What the creator makes (e.g. "Creating Philosophy Videos") |
| `currency` | Currency code (USD / EUR / GBP / …) |
| `patronCount` | Total patron count (free + paid) |
| `paidMemberCount` | Paid-member count |
| `creationCount` | Total post count |
| `payPerName` | "month" / "video" / "creation" |
| `isMonthly` | `true` if billing is monthly |
| `isNsfw` | `true` for adult content |
| `offersFreeMembership` | Whether a free tier exists |
| `offersPaidMembership` | Whether paid tiers exist |
| `publishedAt` | ISO 8601 launch date |
| `avatarPhotoUrl` | Avatar image URL |
| `coverPhotoUrl` | Banner image URL |
| `pledgeUrl` | Direct checkout URL |
| `creator` | `{fullName, vanity, url}` for the underlying user |
| `tiers` | Array of paid tiers (sorted by price ascending) |
| `socialLinks` | `{appName: profileUrl}` for connected social accounts the creator has linked |
| `scrapedAt` | ISO 8601 UTC timestamp |

`mode=search` and `mode=explore` return a lighter creator record instead (`recordType: "searchResult"` / `"exploreCard"`) — only `id`, `name`, `vanity`, `url`, `creationName`, `oneLiner`, `summary`, `patronCount`, `avatarPhotoUrl`, `coverPhotoUrl`, `isNsfw` and (explore only) `category`; `tiers`, `creator`, `socialLinks`, `currency`, `paidMemberCount`, `creationCount`, `publishedAt` and `pledgeUrl` aren't part of those thinner listing payloads.

Each tier in `tiers`:

| Field | Description |
| --- | --- |
| `id` | Tier (reward) ID |
| `title` | Tier title |
| `description` | Tier description (plain text) |
| `amountCents` | Price in cents |
| `amountUsd` / `amountDollars` | Price as a decimal USD/local-currency value (same value, two keys) |
| `currency` | Currency code |
| `patronCount` | Patrons on this tier |
| `requiresShipping` | `true` if physical fulfilment required |
| `imageUrl` | Tier illustration URL |

## Output per post

| Field | Description |
| --- | --- |
| `id` | Post ID |
| `vanity` | Creator vanity (added by us) |
| `title` | Post title |
| `publishedAt` | ISO 8601 publish timestamp |
| `url` | Public post URL |
| `postType` | `video_external_file`, `image_file`, `text_only`, `audio_file`, `link`, … |
| `isPaid` | `true` if patron-only |
| `isPublic` | `true` if visible to anonymous users |
| `patronCount` | Number of patrons who can see the post |
| `commentCount` | Comment count |
| `likeCount` | Like count |
| `teaserText` | Plain-text teaser (when paid post has a public teaser) |
| `content` | Plain-text body (when public) |
| `body` | Same plain-text body, truncated to 5000 characters |
| `thumbnailUrl` | Thumbnail image URL |

## Output per failed input

When a creator slug can't be resolved to a campaign, the actor emits a typed record instead of skipping silently:

| Field | Description |
| --- | --- |
| `recordType` | `"error"` |
| `vanity` | The input slug that failed |
| `reason` | `fan_profile_not_creator` (slug belongs to a patron, not a creator) or `redirect_failed_no_campaign` (no campaign could be found) |
| `scrapedAt` | ISO 8601 UTC timestamp |

Empty fields are dropped from every record at every depth.

## Input

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `mode` | Enum | `creator` | `creator` (campaign + tiers + posts) / `posts` (posts only) / `search` (creators by keyword) / `explore` (browse by category) / `postByUrl` (single post by URL) |
| `creators` | Array | `["PhilosophyTube"]` | Vanity slugs or full URLs (mode=creator/posts) |
| `searchQuery` | String | — | Keyword for `mode=search` |
| `category` | Enum | — | Patreon Explore category — `podcasts`, `video-film`, `music`, `visual-arts`, `comedy`, `games`, `writing`, `drawing-painting`, `comics-graphic-novels`, `tabletop-games`, `science`, `education`, `crafts-diy`, `lifestyle` |
| `postUrls` | Array | — | Patreon post URLs (`https://www.patreon.com/posts/<id>`) for `mode=postByUrl` |
| `includePosts` | Boolean | `true` | When `mode=creator`, also fetch posts |
| `maxPostsPerCreator` | Integer | `20` | Hard cap per-creator post emission (1-200) |
| `minPatronCount` | Integer | — | Drop creators with fewer patrons |
| `excludeNsfw` | Boolean | `false` | Drop NSFW creators |
| `maxItems` | Integer | `50` | Hard cap on total emitted records (1-5000) |
| `autoEscalateOnBlock` | Boolean | `true` | When enabled, the actor automatically uses Apify Proxy if a request is blocked, instead of failing that request. |
| `proxyGroups` | Array | — | Apify proxy groups to use when a block triggers `autoEscalateOnBlock`. Leave empty for the default group. |

### Example input — single creator + posts

```json
{
  "mode": "creator",
  "creators": ["PhilosophyTube"],
  "includePosts": true,
  "maxPostsPerCreator": 20
}
```

### Example input — batch creators

```json
{
  "mode": "creator",
  "creators": ["jordanbpeterson", "PhilosophyTube"],
  "includePosts": false,
  "minPatronCount": 1000
}
```

### Example input — posts only

```json
{
  "mode": "posts",
  "creators": ["PhilosophyTube"],
  "maxPostsPerCreator": 100
}
```

### Example input — search creators

```json
{
  "mode": "search",
  "searchQuery": "philosophy",
  "minPatronCount": 500,
  "maxItems": 50
}
```

### Example input — explore by category

```json
{
  "mode": "explore",
  "category": "podcasts",
  "maxItems": 50
}
```

### Example input — fetch post by URL

```json
{
  "mode": "postByUrl",
  "postUrls": [
    "https://www.patreon.com/posts/85614999",
    "https://www.patreon.com/posts/welcome-to-the-12345"
  ]
}
```

## Example output

```json
{
  "recordType": "creator",
  "id": "114875",
  "vanity": "PhilosophyTube",
  "url": "https://www.patreon.com/PhilosophyTube",
  "name": "Philosophy Tube",
  "creationName": "Creating Philosophy Videos",
  "oneLiner": "Giving away a philosophy degree one video at a time.",
  "summary": "When the British government tripled university tuition in 2010 …",
  "currency": "USD",
  "patronCount": 15382,
  "paidMemberCount": 5233,
  "creationCount": 756,
  "payPerName": "month",
  "offersFreeMembership": true,
  "offersPaidMembership": true,
  "publishedAt": "2014-09-21T17:34:46.000+00:00",
  "creator": {
    "fullName": "Philosophy Tube",
    "vanity": "PhilosophyTube",
    "url": "https://www.patreon.com/PhilosophyTube"
  },
  "tiers": [
    { "id": "...", "title": "Patron", "amountCents": 200, "amountUsd": 2.00, "currency": "USD" },
    { "id": "...", "title": "Producer", "amountCents": 500, "amountUsd": 5.00 }
  ],
  "scrapedAt": "2026-05-06T05:42:18Z"
}
```

## Use cases

- **Creator-economy analytics** — Track patron growth and tier pricing across a portfolio of creators.
- **Brand-deal targeting** — Identify creators with N+ patrons in a niche for sponsorship outreach.
- **Competitive research** — Compare your creator's pricing/post-cadence with peers.
- **Investor due-diligence** — Patron counts are a public approximation of MRR; compare across creators.
- **Content audits** — Pull every public post from a Patreon creator for analysis or archival.

## FAQ

**Do I need a Patreon account?**
No. Every endpoint this actor uses is part of Patreon's public-facing site (the campaign page that any visitor can see) and the public posts JSON API.

**Why does `paidMemberCount` differ from `patronCount`?**
`patronCount` includes free-tier members. `paidMemberCount` is paying patrons only — usually a much smaller number for creators that offer a free tier.

**Are full post bodies included?**
Only for posts marked `is_public=true`. Patron-only posts return a public teaser (when set) but the full body is locked. Each post's `isPaid` / `isPublic` flags tell you which type you have.

**How many posts can I scrape per creator?**
Up to 200 per run via `maxPostsPerCreator`. The Patreon posts API paginates 20 at a time — large pulls take ~1 second per page.

**How current is the data?**
Live — every run hits Patreon at request time. Schedule the actor for hourly / daily refreshes to track patron-growth curves.

**Do I need to set up a proxy?**
No, proxy use is optional. `autoEscalateOnBlock` is on by default so the actor only reaches for Apify Proxy if a request gets blocked; most runs never need it.

## Limitations

- Patron-only post bodies are not accessible without a logged-in account.
- Some creators hide `patronCount` (set `member_count_preference` to TIER_BREAKDOWN); those records will still emit but with the count omitted.
- Earnings figures are never surfaced, even for creators who publish them on their page.
- Comments are not included in this version (each post would need a follow-up fetch); we surface `commentCount` for sentiment-trend approximation.
