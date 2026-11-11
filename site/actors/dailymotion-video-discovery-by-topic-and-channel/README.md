# Dailymotion Video Discovery by Topic & Channel

Dailymotion's catalog is organized into keyword search, trending feeds, 17 topic categories, channel pages and playlists. This actor reads whichever of those you point it at — plus specific video IDs and a channel/user keyword search — and returns one clean JSON record per video or channel, straight from Dailymotion's public API. No login, no API key, no proxy.

## What this actor does

- **Seven modes:** `search` videos by keyword, browse `trending`, walk a `byTopic` category, pull a `byUser` channel's uploads, read a `byPlaylist`, look up `byVideoIds`, or `searchUsers` for Dailymotion channels and creators.
- **Filters that match how the API really behaves:** language, country (search/trending), publish date range, tag, verified channels only, live streams only, and exact min/max duration.
- **More than 1,000 results per query:** setting `maxItems` above 1,000 makes the actor sweep Dailymotion's date windows automatically (newest-first or oldest-first), deduplicating as it goes — up to 10,000 records per run.
- **Engagement over time:** total views plus views in the last hour, day, week and month, so you can see velocity, not just the all-time number.
- **Full channel context per video:** owner ID, display name, profile URL, followers, following, total videos and playlists, country, language, join date, avatar, cover and verification.
- **Geo-blocking, hashtags and every thumbnail size** are returned when Dailymotion has them; fields the source doesn't carry are simply left out of that record.

## Output per video

| Field | Description |
|---|---|
| `videoId` | Dailymotion video ID. |
| `title` / `description` | Video title and description. |
| `ownerId` / `ownerUsername` / `ownerUrl` | Channel ID, display name and profile URL. |
| `ownerVideosTotal` / `ownerFollowers` / `ownerFollowing` / `ownerPlaylistsTotal` | Channel counters. |
| `ownerCountry` / `ownerLanguage` / `ownerCreatedAt` | Channel country, language and join date (ISO 8601). |
| `ownerVerified` / `ownerPartner` | Channel verification and partner status. |
| `ownerAvatarUrl` / `ownerAvatarUrlLarge` / `ownerCoverUrl` | Channel avatar (360/720 px) and cover image. |
| `views` | Total views. |
| `viewsLastHour` / `viewsLastDay` / `viewsLastWeek` / `viewsLastMonth` | Recent view counts (velocity). |
| `likes` / `bookmarks` | Like and bookmark counts. |
| `comments` | Comment count. Dailymotion no longer shows comments, so this is always 0 — kept for compatibility. |
| `duration` | Duration in seconds. |
| `publishedAt` / `updatedAt` / `uploadedAt` | Publish, last-update and upload timestamps (ISO 8601 UTC). |
| `thumbnailUrl` / `thumbnailUrlSmall` / `thumbnailUrlLarge` / `thumbnailUrlMax` | Thumbnails (default, 240 px, 720 px, 1080 px). |
| `firstFrameUrl` | 720 px first-frame image. |
| `videoUrl` / `tinyUrl` | Canonical video page and dai.ly short link. |
| `embedUrl` / `embedHtml` | Embeddable player URL and iframe snippet. |
| `allowEmbed` / `allowedInPlaylists` | Embedding and playlist permissions. |
| `width` / `height` / `aspectRatio` | Video dimensions and frame aspect ratio. |
| `availableFormats` | Available stream qualities, e.g. `sd`, `hq`, `hd720`, `hd1080`. |
| `tags` / `hashtags` | Searchable tags and uploader hashtags. |
| `channel` / `channelName` | Dailymotion category ID and label. |
| `language` / `country` | Video language and country codes. |
| `explicit` / `isCreatedForKids` | Content flags when set. |
| `mode` | `vod` for on-demand, `live` for livestreams. |
| `geoBlockingMode` / `geoBlockingCountries` | Geo-restriction mode (`allow`/`deny`) and the affected country codes. |
| `sourceUrl` | The search/topic/channel URL this record was found through, when the mode used one. |
| `recordType` | Always `"video"`. |
| `scrapedAt` | ISO 8601 timestamp of when the record was fetched. |

## Output per channel (mode = `searchUsers`)

| Field | Description |
|---|---|
| `channelId` | Dailymotion user/channel ID. |
| `username` / `screenname` | Handle and display name. |
| `description` | Channel bio, when present. |
| `verified` / `partner` | Verification and partner status. |
| `url` | Channel profile URL. |
| `followers` / `following` | Follower and following counts. |
| `videos` / `playlists` / `views` / `reposts` | Catalog and engagement totals. |
| `country` / `language` | Channel country and language. |
| `createdAt` | Join date (ISO 8601). |
| `avatarUrl` / `avatarUrlLarge` / `coverUrl` | Avatar (360/720 px) and cover image. |
| `sourceUrl` / `recordType` / `scrapedAt` | Provenance (`recordType` is `"channel"`) and fetch time. |

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string (select) | `search` | `search` / `trending` / `byTopic` / `byUser` / `byPlaylist` / `byVideoIds` / `searchUsers`. |
| `searchQueries` | array | – | One or more keywords (mode=search, searchUsers). The daily test uses `["technology news"]`. |
| `usernames` | array | `[]` | Channel usernames, profile URLs or IDs (mode=byUser). Multiple allowed. |
| `topic` | string (select) | `news` | One of Dailymotion's 17 categories (mode=byTopic). |
| `playlists` | array | `[]` | Playlist URLs or IDs (mode=byPlaylist). Multiple allowed. |
| `videoIds` | array | `[]` | Video IDs or URLs (mode=byVideoIds). |
| `sortBy` | string (select) | `relevance` | Result order; see the Sort section below. |
| `language` | string (select) | `""` (all) | Only videos in this language. |
| `country` | string (select) | `""` (all) | Only videos from this country (mode=search, trending). |
| `verifiedOnly` | boolean | `false` | Only videos from verified channels (mode=search, trending). |
| `liveOnly` | boolean | `false` | Only live streams. |
| `tag` | string | – | Only videos carrying this tag (mode=search, byTopic, byUser, byPlaylist). |
| `createdAfter` / `createdBefore` | string | – | Publish date window, `YYYY-MM-DD` (mode=search, trending, byTopic, byUser). Long ranges are fetched automatically in overlapping windows. |
| `minDuration` / `maxDuration` | integer | – | Only videos within this duration range, in seconds. |
| `maxItems` | integer | `20` | Maximum records to return (1–10,000). Values above 1,000 use automatic date-window partitioning in search/topic/channel modes. |

### Sort order values

| Value | Works in | Notes |
|---|---|---|
| `relevance` | search, searchUsers | Default for search. |
| `recent` | every video mode | Newest first. |
| `trending` | every video mode | Dailymotion's trending order. |
| `visited` / `visited-hour` / `visited-today` / `visited-week` / `visited-month` | every video mode | Most-viewed windows. |
| `random` | every video mode | Random sample. |
| `old` / `id-asc` | search, byTopic, byUser, byPlaylist | Oldest first. |
| `popular` | searchUsers | Most-followed channels. |

Picking a value a mode can't honor (e.g. `relevance` in `trending`) falls back to that mode's default order, and the run's status message tells you it did. Runs with `maxItems` above 1,000 always use a `recent` or `old` time-ordered sweep (the status message says so); `byVideoIds` keeps your input order.

### Example: keyword search

```json
{
    "mode": "search",
    "searchQueries": ["electric cars", "solar power"],
    "sortBy": "visited-week",
    "language": "en",
    "maxItems": 100
}
```

### Example: a large channel sweep (more than 1,000 videos)

```json
{
    "mode": "byUser",
    "usernames": ["euronews-en"],
    "sortBy": "recent",
    "maxItems": 8000
}
```

The actor walks the channel's history in automatically-sized date windows and removes overlaps, so the dataset contains 8,000 distinct videos (or every video the channel has, whichever comes first).

### Example: a channel's uploads, by profile URL

```json
{
    "mode": "byUser",
    "username": "https://www.dailymotion.com/user/euronews-en",
    "createdAfter": "2026-01-01",
    "maxItems": 200
}
```

### Example: a playlist

```json
{
    "mode": "byPlaylist",
    "playlist": "https://www.dailymotion.com/playlist/x6pu6c",
    "maxItems": 50
}
```

### Example: specific videos by ID

```json
{
    "mode": "byVideoIds",
    "videoIds": ["x4d4447", "https://www.dailymotion.com/video/x3gmshg"],
    "maxItems": 10
}
```

### Example: channels and creators by keyword

```json
{
    "mode": "searchUsers",
    "searchQueries": ["space news"],
    "sortBy": "popular",
    "maxItems": 30
}
```

## Use cases

- **Channel monitoring** — track a publisher's new uploads and their first-day view velocity.
- **Topic and trend research** — compare the most-viewed videos across categories and languages.
- **Competitive benchmarking** — collect follower, catalog and engagement counts for a set of channels.
- **Media analysis** — build duration-bucketed datasets of short clips vs long-form content.
- **Archival and licensing research** — map playlists, creators and geo-restrictions across a catalog.
- **Dashboards and alerting** — feed fresh upload metadata into BI tools on a schedule.

## FAQ

**Do I need a Dailymotion account or API key — and will this keep working when Dailymotion moves to API v2?**
No account or key is needed: the actor reads Dailymotion's public read API without credentials, cookies or proxy setup. That API is still documented and supported (no sunset announced as of 2026-10). API v2 targets authenticated publisher workflows and requires private API keys, so a credential-free actor cannot use it; if Dailymotion ever announces a sunset for the public read API, this actor will need a rework. Current behaviour is tracked in the project research notes.

**How many records can one run return?**
Up to 10,000 per run. Dailymotion's API itself returns at most 1,000 results per query, so the actor fetches more by sweeping the query's date range in overlapping windows (newest-first or oldest-first) and removing overlaps — the run's status message tells you how many windows were used. `byPlaylist` and `searchUsers` modes stay at 1,000 per query, because the playlist and user-search endpoints don't accept date filters. The date filters are applied by Dailymotion in roughly five-minute buckets, so at a day boundary a window can include up to five minutes of neighbouring content; the actor never fabricates or drops records because of it and reports what the source returned.

**Which modes, filters and sort orders does Dailymotion's API actually support?**
Playlist search by keyword is not available — the public API supports keyword search only for videos and channels/users (the actor reads playlists by URL or ID, `mode=byPlaylist`). The country filter is accepted only together with a keyword search or the trending order; elsewhere it is ignored and the run's status message says so. Likewise, the `old`/`id-asc` orders are rejected for the plain trending feed and for keyword searches outside a mode that supports them — the actor falls back to the mode's default order and notes it in the status message.

**Why do some videos have no `tags`, `description`, `firstFrameUrl`, or channel no `ownerCoverUrl`?**
Dailymotion only stores those fields for some uploads, and those artifacts genuinely don't exist for every video or channel — the player metadata returns empty first-frame sets for the same videos, and channels without a cover get a generated gradient on the site. When a field is missing from the source it is left out of the record entirely: records never contain empty placeholders and nothing is faked.

**Why is `comments` always 0?**
Dailymotion removed public comments from its platform; the API reports 0 for every video and the comments endpoint returns an empty list (re-verified 2026-10-17 across 300 videos). The field is kept so existing integrations don't break.

**What do the `viewsLastHour` / `viewsLastDay` / `viewsLastWeek` / `viewsLastMonth` fields mean — and are the numbers accurate?**
They are Dailymotion's own rolling view counters for the 60 minutes / 24 hours / 7 days / 30 days before the request, and the actor reads the video's metrics from the same API the video page uses. They usually nest inside each other, but because Dailymotion updates the total and window counters independently, a window counter can occasionally exceed `views` by a count or two; the actor reports the API's numbers as-is rather than correcting them.

**Can I download videos or get direct stream URLs?**
No. This actor is for discovery and analytics: it returns metadata, thumbnails and embed links, not media files. Stream/download URLs require authenticated publisher access that Dailymotion doesn't grant to public integrations.

**What topics can I browse?**
All 17 built-in categories: News, Sports, Music, Tech, Animals, Cars, Celeb, Comedy & Entertainment, Creative, Education, Gaming, Kids, Lifestyle & How-to, Movies, Travel, TV and Webcam.

**Are private or unlisted videos included?**
No — only publicly accessible videos are returned. Unavailable IDs passed to `byVideoIds` are skipped, and the run status reports how many could not be accessed.

**Is this affiliated with Dailymotion?**
No. This actor is an independent tool that reads Dailymotion's public API and is not affiliated with or endorsed by Dailymotion.
