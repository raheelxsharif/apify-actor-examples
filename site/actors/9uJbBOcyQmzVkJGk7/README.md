# Snapchat Publisher & Show Episode Finder

Snapchat's Discover publishers and Snap Shows — news outlets, entertainment channels, and episodic series — each have a public profile page with episode listings, season structure, and Spotlight content. This actor reads that page for any publisher or show you give it and returns a structured profile record.

## What this actor does

- **Fetches the public publisher profile** for any Snapchat publisher page or Snap Show
- **Lists premium stories and episodes** with episode and season numbers, when available
- **Lists show season structure** with episode counts per season
- **Lists publisher Spotlight cards** with direct media URLs and engagement stats
- **Empty fields are omitted** from each record instead of being returned as null

## Output per publisher

- `publisherId` — businessProfileId UUID
- `pageUrl` — full `snapchat.com/p/{id}` URL
- `displayName`, `username` — publisher or show display name and handle, when the page exposes one
- `bio` — publisher description
- `isVerified` — whether the publisher has a verified badge
- `subscriberCount` — total subscriber count
- `profilePictureUrl` — publisher logo/avatar image
- `snapcodeImageUrl` — publisher Snapcode image
- `squareHeroImageUrl` — banner/hero image
- `websiteUrl` — external website link
- `category`, `subcategory` — publisher category and subcategory
- `address` — publisher's published address, when set
- `mutableName`, `hostUserId` — internal profile name and host account ID, when exposed
- `primaryColor` — brand hex color (e.g. `#ff5831`)
- `publisherType` — `SHOW` or `PUBLISHER`
- `hasStory`, `hasCuratedHighlights`, `hasSpotlightHighlights`, `shouldHideUsername` — profile flags
- `premiumStories[]` — when `includeStories` is enabled: `storyId`, `storyTitle`, `storyType`, `thumbnailUrl`, `episodeNumber`, `seasonNumber`, `postedAt`
- `preselectedStory` — when `includeStories` is enabled and the page features one: `storyId`, `storyTitle`, `episodeNumber`, `seasonNumber`, `snaps[]` (`snapIndex`, `mediaUrl`, `mediaType`, `snapId`, `mediaPreviewUrl`) — the only place direct premium-episode media URLs appear
- `premiumShowSeasons[]` — when `includeShowSeasons` is enabled: `seasonId`, `seasonNumber`, `episodeCount`
- `publisherSpotlights[]` — when `includeSpotlights` is enabled: `storyId`, `compositeStoryId`, `snapId`, `snapUrl`, `thumbnailUrl`, `videoUrl`, `imageUrl`, `durationSeconds`, `width`, `height`, `creatorUsername`, `creatorDisplayName`, `creatorFollowerCount`, `description`, `caption`, `lat`, `lng`, `geoDisplayName`, `engagementStats` (`viewCount`, `shareCount`, `boostCount`, `commentCount`, `pendingCommentCount`, `recommendCount`, `remixCount`), `createdAt`, `latestUpdatedAt`
- `scrapedAt` — ISO 8601 scrape timestamp

If a publisher ID or URL can't be resolved, that entry instead becomes an error record with `input`, `error`, and `scrapedAt`.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `publisherUrlsOrIds` | array (required) | – | Publisher page URLs (`snapchat.com/p/{id}`, `story.snapchat.com/p/{id}`) or bare businessProfileId UUIDs |
| `includeStories` | boolean | `true` | Extract the premium story/episode list |
| `includeShowSeasons` | boolean | `true` | Extract the show season list |
| `includeSpotlights` | boolean | `true` | Extract publisher Spotlight story cards |
| `proxyConfiguration` | object | – | Optional proxy configuration; publisher pages are public and don't require one |

### Example: a single Snap Show, full detail

```json
{
  "publisherUrlsOrIds": ["5551edcb-8d47-489c-aa13-7aecfedc814e"],
  "includeStories": true,
  "includeShowSeasons": true,
  "includeSpotlights": true
}
```

### Example: multiple publishers, profile only

```json
{
  "publisherUrlsOrIds": [
    "https://www.snapchat.com/p/5551edcb-8d47-489c-aa13-7aecfedc814e",
    "https://story.snapchat.com/p/efd8a13d-b69a-41d0-9c04-d19f87829798"
  ],
  "includeStories": false,
  "includeShowSeasons": false,
  "includeSpotlights": false
}
```

## Use cases

- **Media monitoring** — track a news outlet or entertainment channel's subscriber count and output over time
- **Content strategy research** — study season and episode structure across competing Snap Shows
- **Spotlight performance tracking** — pull engagement stats for a publisher's Spotlight content
- **Partnership scouting** — check subscriber counts and categories before reaching out to a publisher for collaboration
- **Catalog building** — maintain a reference list of publisher profiles, logos, and categories

## FAQ

**Is this affiliated with Snapchat?**
No. This is an independent, third-party actor that reads publicly available Snapchat publisher pages; it isn't operated by or affiliated with Snapchat.

**What's the difference between a publisher and a Snap Show?**
A Snap Show is an episodic series (seasons, numbered episodes); a publisher is a media outlet that posts individual stories without episode structure. Both live at `snapchat.com/p/{id}`, and `publisherType` tells you which kind a given profile is.

**Where do I find a businessProfileId?**
It's the UUID in the publisher page URL: `snapchat.com/p/{businessProfileId}`.

**Why is `displayName` populated but there's no username field?**
Many Snap Shows and publisher accounts are registered as business profiles without a traditional Snapchat username. `displayName` always carries the visible name shown on the page.

**Why does an invalid publisher ID return an error record instead of failing the run?**
Regular personal accounts and invalid IDs don't resolve to a publisher page, so that entry is recorded as an error row (`input`, `error`, `scrapedAt`) and the run continues to the next one.

**Can I get the actual video content of premium episodes?**
`premiumStories[]` only carries episode thumbnails and metadata, but the page's featured episode (`preselectedStory`) includes direct snap media URLs when Snapchat marks one as preselected. Publisher Spotlight content always includes a direct `videoUrl`.

**Do I need a proxy?**
No. Publisher pages are publicly accessible without one; a proxy may help only on very high-volume runs.

**How many publishers can I look up in one run?**
As many as you add to `publisherUrlsOrIds` — each is fetched and produces one record (or one error record).
