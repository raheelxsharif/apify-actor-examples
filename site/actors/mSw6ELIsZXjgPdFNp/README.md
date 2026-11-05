# YouTube Hashtag Video & Shorts Ranking Scraper

Every YouTube hashtag page ranks the videos and Shorts currently tagged with it. This actor reads that ranking for one or more hashtags and returns each video or Short in position order, with its view count, channel, and (for regular videos) duration and publish time — no login or cookies needed.

## What this actor does

- **Looks up one or more hashtags** at once, with or without the `#` prefix
- **Returns both regular videos and Shorts**, or either one alone
- **Preserves ranking order** — each result carries its 1-indexed position on the hashtag page
- **Reports the hashtag's own totals** — total video count and total channel count, when YouTube's page exposes them
- **Localizes results** to a chosen country/region
- **Empty fields are omitted** — a record only contains the values that were actually present for that item

## Output per video or Short

| Field | Description |
|---|---|
| `hashtag` | Normalized hashtag, without the `#` prefix |
| `hashtagTitle` | The hashtag page's own display title, when YouTube exposes one |
| `videoCountText` / `videoCount` | Total videos tagged with this hashtag, as displayed text and as a parsed number |
| `channelCountText` / `channelCount` | Total channels that have posted under this hashtag, as displayed text and as a parsed number |
| `position` | 1-indexed rank within this hashtag's results |
| `videoId` | YouTube video ID |
| `title` | Video title |
| `channelName` | Name of the channel that posted it |
| `channelId` | YouTube channel ID |
| `channelUrl` | Full URL to the channel page |
| `viewCountText` | View count as displayed, e.g. `1.2M views` |
| `viewCount` | View count as an exact number, when YouTube shows an unabridged count rather than a rounded label |
| `publishedTimeText` | Relative publish time, e.g. `3 days ago` (regular videos only) |
| `durationText` | Video duration, e.g. `12:34` (regular videos only) |
| `durationSeconds` | Video duration in seconds, parsed from `durationText` (regular videos only) |
| `thumbnailUrl` | URL of the highest-resolution thumbnail available |
| `isShort` | `true` for a YouTube Short, `false` for a regular video |
| `badges` | Content badges when present, e.g. `["CC", "4K"]` |
| `scrapedAt` | ISO 8601 UTC timestamp of the run |

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `hashtags` | array | – | Hashtags to scrape, with or without the `#` prefix (required) |
| `maxResultsPerHashtag` | integer | `50` | Maximum videos/Shorts to return per hashtag (1–500) |
| `contentType` | string | `all` | `all`, `videos` (regular videos only), or `shorts` (Shorts only) |
| `market` | string | `US` | Country/region for localized ranking, e.g. `US`, `GB`, `DE`, `JP`, `IN`, `BR` |

### Example: single hashtag, all content types

```json
{
  "hashtags": ["#webscraping"],
  "maxResultsPerHashtag": 100,
  "market": "US"
}
```

### Example: multiple hashtags, Shorts only

```json
{
  "hashtags": ["#travel", "#food"],
  "contentType": "shorts",
  "maxResultsPerHashtag": 50
}
```

### Example: regular videos localized to Japan

```json
{
  "hashtags": ["anime"],
  "contentType": "videos",
  "market": "JP",
  "maxResultsPerHashtag": 200
}
```

## Use cases

- **Trend discovery** — see which videos and Shorts are currently ranking under a topic hashtag
- **Influencer sourcing** — spot creators who repeatedly rank under a niche hashtag
- **Campaign tracking** — monitor a branded hashtag's ranked videos over time
- **Content research** — compare view counts and formats (video vs Short) across a topic
- **Regional analysis** — compare how a hashtag ranks differently across markets with the `market` input

## FAQ

**Do I need a YouTube account, API key, or cookies?**
No. Hashtag pages are public, and this actor reads them without any authentication.

**Why are some fields missing from Shorts results?**
YouTube's Shorts cards don't carry the same metadata as regular video cards — notably no `publishedTimeText` or `durationText`. Those fields are simply left out rather than filled with a placeholder.

**How many results can I get per hashtag?**
Up to `maxResultsPerHashtag` (maximum 500), limited by how many videos the hashtag page actually ranks.

**What does `market` affect?**
It changes the region/language used when loading the hashtag page, which can shift which videos rank and how counts/text are formatted.

**Can I scrape several hashtags in one run?**
Yes — list them all under `hashtags`; the actor processes each one and combines the results into a single dataset.

**What's the difference between the three `contentType` options?**
`all` returns both regular videos and Shorts, `videos` returns only regular videos, and `shorts` returns only Shorts.

**What happens if a hashtag has no results?**
The run continues to the next hashtag in the list rather than failing.
