# Snapchat Publisher & Show Episode Finder

Read **Snapchat** publisher and Snap Show catalogues without logging in. Give the actor a publisher page URL, a bare business profile ID, or an episode URL and it returns the public profile, the latest page stories, every season, the full episode history, optional HLS media, and Spotlight cards. A Discover mode reads the current country feed with cursor-style pagination. HTTP-only, no cookies, no proxy setup.

## What this actor does

- **Three modes:** `byPublisher` for a profile and episode catalogue, `byEpisodeUrl` for one episode's snaps and HLS track, and `discover` for the country/locale feed
- **Complete episode history:** walks every season with Snapchat's public `PublishedPremiumStories` cursor, not just the 13-14 stories embedded in the page
- **Optional episode media:** adds the episode-level HLS `videoTrackUrl`, snap count and per-snap preview records
- **Publisher Spotlights:** media URLs, creator details and engagement counters from the profile page
- **Discover feed:** country and locale aware, paginated by tap IDs until the feed is exhausted or `maxItems` is reached
- **Empty fields are omitted:** no nulls, no empty strings, no empty arrays and no 1970 placeholder dates

## Output per publisher (`mode = byPublisher`)

- `recordType: "publisher"`, `publisherId`, `pageUrl`
- `displayName`, `username` — visible name and handle when the profile exposes one
- `isVerified`, `subscriberCount`, `bio`
- `profilePictureUrl`, `snapcodeImageUrl`, `squareHeroImageUrl`
- `websiteUrl`, `category`, `subcategory`, `publisherType` (`SHOW` or `NORMAL` on publisher pages)
- `primaryColor`, `hostUserId`, `address`, `mutableName`
- `hasStory`, `hasCuratedHighlights`, `hasSpotlightHighlights`, `shouldHideUsername`
- `seasonCount`, `episodeCount`
- `premiumShowSeasons[]` — `seasonId`, `seasonNumber`, `episodeCount` declared by the page
- `episodes[]` — `episodeId`, `episodeUrl`, `title`, `thumbnailUrl`, `publishedAt`, `seasonId`, `seasonNumber`, `episodeNumber`, plus `videoTrackUrl`, `snapCount` and `snaps[]` when media is enabled
- `premiumStories[]` — the latest stories embedded in the page, kept for compatibility: `storyId`, `storyTitle`, `storyType`, `thumbnailUrl`, `episodeNumber`, `seasonNumber`, `postedAt`
- `preselectedStory` — the featured episode, with `storyId`, `storyTitle`, `episodeNumber`, `seasonNumber`, `episodeUrl` and `snaps[]` when available
- `publisherSpotlights[]` — `storyId`, `compositeStoryId`, `snapId`, `snapUrl`, `thumbnailUrl`, `videoUrl`, `imageUrl`, `durationSeconds`, `width`, `height`, `creatorUsername`, `creatorDisplayName`, `creatorFollowerCount`, `description`, `caption`, `lat`, `lng`, `geoDisplayName`, `engagementStats`, `createdAt`, `latestUpdatedAt`
- `scrapedAt`

## Output per episode (`mode = byEpisodeUrl`)

- `recordType: "episode"`, `episodeId`, `episodeUrl`, `publisherId`
- `displayName`, `username`, `subscriberCount`, `isVerified`, `profilePictureUrl`, `squareHeroImageUrl` — publisher context
- `title`, `thumbnailUrl`, `publishedAt`, `episodeNumber`, `seasonNumber`
- `videoTrackUrl` — the episode-level HLS track
- `snapCount`, `snaps[]` — `snapId`, `index`, `mediaType`, `mediaUrl` when public, `previewUrl`, `intervalStartMs`
- `scrapedAt`

## Output per Discover story (`mode = discover`)

- `recordType: "discoverStory"`, `storyId`, `storyType`, `title`, `publishedAt`, `tapId`
- `thumbnailUrl`, `videoTrackUrl` when the feed supplies one
- `pageUrl`, `publisherId`, `displayName`, `username`, `bio`, `profilePictureUrl`, `squareHeroImageUrl`, `snapcodeImageUrl`, `primaryColor`, `isVerified`
- `snapCount`, `snaps[]`
- `scrapedAt`

Rows that cannot be resolved are returned as `recordType: "error"` with `input`, optional `publisherId` / `episodeId`, `statusCode`, `error` and `scrapedAt`.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `byPublisher` | `byPublisher` / `byEpisodeUrl` / `discover` |
| `publisherUrlsOrIds` | array | – | Publisher URLs, `story.snapchat.com/p/{id}` links or bare UUIDs (mode=byPublisher) |
| `episodeUrls` | array | – | `snapchat.com/p/{publisherId}/{episodeId}` URLs (mode=byEpisodeUrl) |
| `country` | string | `US` | ISO country code used for episode requests and the Discover feed |
| `locale` | string | `en-US` | Language-region tag for Discover, e.g. `en-GB` or `de-DE` |
| `includeStories` | boolean | `true` | Walk every season and fill `episodes[]` (mode=byPublisher); the latest page stories stay in `premiumStories[]` either way |
| `includeShowSeasons` | boolean | `true` | Include `premiumShowSeasons[]` (mode=byPublisher) |
| `includeSpotlights` | boolean | `true` | Include `publisherSpotlights[]` (mode=byPublisher) |
| `includeEpisodeMedia` | boolean | `false` | Fetch HLS tracks and snaps for every returned episode (mode=byPublisher) |
| `maxEpisodesPerPublisher` | integer | `100` | Hard cap on `episodes[]` per publisher (1–5000) |
| `maxItems` | integer | `100` | Hard cap on dataset rows, including error rows (1–5000) |

### Example: publisher profile with the latest page data only

```json
{
  "mode": "byPublisher",
  "publisherUrlsOrIds": ["efd8a13d-b69a-41d0-9c04-d19f87829798"],
  "includeStories": false,
  "includeShowSeasons": false,
  "includeSpotlights": false,
  "maxItems": 1
}
```

### Example: complete episode catalogue for a Snap Show

```json
{
  "mode": "byPublisher",
  "publisherUrlsOrIds": ["https://www.snapchat.com/p/f905ada7-32eb-4f9c-8ddf-daa529dcc79f"],
  "includeStories": true,
  "includeShowSeasons": true,
  "includeSpotlights": true,
  "includeEpisodeMedia": false,
  "maxEpisodesPerPublisher": 1000,
  "maxItems": 10
}
```

### Example: one episode with its HLS track and snaps

```json
{
  "mode": "byEpisodeUrl",
  "episodeUrls": [
    "https://www.snapchat.com/p/f905ada7-32eb-4f9c-8ddf-daa529dcc79f/3298969776404480"
  ],
  "country": "US",
  "maxItems": 1
}
```

### Example: the Discover feed for a country

```json
{
  "mode": "discover",
  "country": "GB",
  "locale": "en-GB",
  "maxItems": 100
}
```

## Use cases

- **Media monitoring** — track subscriber counts, season structure and publishing cadence for news outlets and entertainment channels
- **Show catalogues** — build a complete season and episode index for a Snap Show instead of the handful of episodes on the page
- **Episode archives** — capture every episode URL, title, thumbnail, publication time and HLS track for downstream processing
- **Content strategy research** — compare episode volume and Spotlight performance across publishers
- **Discover trend tracking** — sample the current Discover feed per country and locale for editorial or partnership research
- **Partnership scouting** — check publisher categories, websites and audience size before outreach

## FAQ

**Is this affiliated with Snapchat?**
No. This is an independent third-party actor that reads publicly accessible Snapchat pages and feed responses. It is not operated by or affiliated with Snapchat.

**What is the difference between a publisher and a Snap Show?**
A Snap Show is an episodic series with seasons and numbered episodes. A publisher is a media outlet or creator profile that may publish stories without a season structure. Both use `snapchat.com/p/{id}`, and `publisherType` tells you which kind of profile the page represents.

**Where do I find a publisher ID?**
It is the UUID in the profile URL, for example `snapchat.com/p/{businessProfileId}`. A bare UUID is also accepted.

**Why does `episodes[]` contain more episodes than `premiumStories[]`?**
`premiumStories[]` is the short list embedded in the profile page for compatibility. `episodes[]` is the full public catalogue assembled season by season, so it can contain hundreds or thousands of rows.

**Why is a season's declared episode count sometimes higher than the number returned?**
Snapchat's season summary can declare more episodes than its public cursor returns. The actor reports the episodes it actually receives in `episodes[]` and keeps the page's declared count in `premiumShowSeasons[]`.

**Why do some snaps have no `mediaUrl`?**
Snapchat only exposes a direct media URL for a small subset of snaps. The episode-level `videoTrackUrl` is the reliable playable media, while `previewUrl` gives the snap thumbnail.

**Why is `durationSeconds` sometimes missing from a Spotlight card?**
Snapchat only sends a duration for some cards. When it is absent the field is omitted rather than set to zero.

**Why does an invalid or removed publisher return an error row?**
Removed publishers, wrong IDs and region-restricted content would otherwise abort the run. The actor records them as typed error rows and continues with the next input.

**Do I need a proxy or a Snapchat account?**
No. The actor works without cookies, logins or proxy setup. Country restrictions can still make some publishers unavailable from a given location, and those rows are reported as HTTP 451 errors.

**How many rows can one run return?**
Use `maxItems` to set the global cap and `maxEpisodesPerPublisher` to bound a single publisher's `episodes[]`. Both are hard limits, and error rows count toward `maxItems`.
