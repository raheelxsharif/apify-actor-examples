# Unsplash Photo Search by Topic, Collection & Creator

Scrape **Unsplash** photos by keyword, curated topic, collection, photographer, photo ID, or any supported Unsplash URL. Each row returns five image sizes, full photographer attribution, dimensions, orientation, dominant colour, BlurHash, likes, and optional tags, camera EXIF, geolocation, topic memberships, and view/download counts. No API key, login, or proxy setup.

## What this actor does

- **Seven ways to source photos:** keyword search, curated topics, numbered collections, photographer portfolios, specific photos, related photos, or a mixed list of pasted Unsplash URLs.
- **Search filters:** orientation, dominant colour, best-match or newest-first ordering, and a free-photos-only switch.
- **Portfolio ordering:** pinned (profile order), newest, oldest, or most popular.
- **Rich metadata:** five image resolutions, photographer name/username/profile/socials, dimensions, colour, BlurHash, likes, and timestamps on every row.
- **Optional detail pass:** adds tags, camera EXIF, place names and coordinates, topic memberships, and real view/download counts for free photos.
- **Optional photographer totals:** each photographer's lifetime views and downloads.
- **Empty fields are omitted** — a photo without EXIF or a tagged location simply has no such field.

## Output per photo

**Identity and content**

- `id`, `slug` — Unsplash photo ID and descriptive slug
- `title`, `description`, `altDescription`, `shortDescription` — captions and alt text when available
- `width`, `height`, `aspectRatio`, `orientation` — native dimensions and `landscape` / `portrait` / `squarish`
- `dominantColor`, `blurHash`, `likes`
- `isPremium`, `premium`, `plus` — Unsplash+ flags
- `datePublished`, `updatedAt`, `promotedAt`

**Images**

- `imageUrl` — regular 1080px image
- `thumbnailUrl` — 200px thumbnail
- `imageUrls.raw`, `imageUrls.full`, `imageUrls.regular`, `imageUrls.small`, `imageUrls.thumb`, `imageUrls.smallS3` — every size in one object (`smallS3` is included for premium photos when the S3 copy responds; the actor checks up to 100 premium photos per details run and falls back to the always-present `plus.unsplash.com` variants when it does not)

**Photographer attribution**

- `photographerId`, `photographerUsername`, `photographerName`, `photographerFirstName`, `photographerLastName`
- `photographerBio`, `photographerLocation`, `photographerPortfolioUrl`, `photographerProfileUrl`
- `photographerInstagram`, `photographerTwitter`, `photographerForHire`
- `photographerTotalPhotos`, `photographerTotalFreePhotos`, `photographerTotalLikes`, `photographerTotalCollections`, `photographerProfileImageUrl`
- `photographerTotalViews`, `photographerTotalDownloads` — when photographer totals are enabled

**Photo details (when enabled)**

- `tags` — up to 20 keyword tags
- `topics` — `{id, slug, title}` memberships
- `exif.make`, `exif.model`, `exif.name`, `exif.exposureTime`, `exif.aperture`, `exif.focalLength`, `exif.iso`
- `location.name`, `location.city`, `location.country`, `location.latitude`, `location.longitude`
- `locationName` — the place name as a flat field
- `views`, `downloads` — free photos only; Unsplash does not publish stats for premium photos

**Licence, downloads, provenance**

- `license` — Unsplash License or Unsplash+ License
- `downloadUrl` — a direct CDN image file that opens without a browser session
- `downloadPageUrl` — Unsplash's tracked download page
- `sourceUrl` — the photo's page on unsplash.com
- `sourceType`, `sourceValue`, `sourceTitle` — which query, topic, collection, photographer, or photo produced the row
- `recordType` — always `photo`
- `scrapedAt` — collection timestamp

## Input

| Field | Type | Default | Description |
| --- | --- | --- | --- |
| `mode` | String (select) | `search` | `search`, `byTopic`, `byCollection`, `byUser`, `byPhoto`, `byRelated`, or `byUrls`. |
| `searchQueries` | Array | `["nature"]` | Keywords to search (mode=search). |
| `topicSlugs` | Array (select) | – | Curated topics (mode=byTopic). |
| `collectionIds` | Array | – | Collection IDs or URLs (mode=byCollection). |
| `usernames` | Array | – | Photographer usernames or profile URLs (mode=byUser). |
| `photoIds` | Array | – | Photo IDs, slugs, or photo URLs (mode=byPhoto or byRelated). |
| `startUrls` | Array | – | Mixed Unsplash search/topic/collection/photographer/photo URLs (mode=byUrls). |
| `orientation` | String (select) | `any` | `any`, `landscape`, `portrait`, or `squarish`. Exact filter in every mode. |
| `color` | String (select) | `any` | Dominant colour for keyword searches only. |
| `freeOnly` | Boolean | false | Exclude Unsplash+ photos. |
| `searchOrderBy` | String (select) | `relevant` | `relevant` or `latest` (mode=search). |
| `userOrderBy` | String (select) | `pinned` | `pinned`, `latest`, `oldest`, or `popular` (mode=byUser). |
| `includePhotoDetails` | Boolean | false | Add tags, EXIF, location, topics, and free-photo stats. |
| `includePhotographerStats` | Boolean | false | Add lifetime views/downloads for each photographer. |
| `maxItems` | Integer | 20 | Maximum photos across all sources (1-1000). |

### Example: keyword search with filters

```json
{
  "mode": "search",
  "searchQueries": ["mountain sunrise", "coastal fog"],
  "orientation": "landscape",
  "color": "blue",
  "freeOnly": true,
  "searchOrderBy": "latest",
  "maxItems": 80
}
```

### Example: a curated topic with details

```json
{
  "mode": "byTopic",
  "topicSlugs": ["nature", "wallpapers"],
  "orientation": "portrait",
  "includePhotoDetails": true,
  "maxItems": 40
}
```

### Example: a collection and a photographer together

```json
{
  "mode": "byUrls",
  "startUrls": [
    { "url": "https://unsplash.com/collections/1065396" },
    { "url": "https://unsplash.com/@v2osk" }
  ],
  "userOrderBy": "latest",
  "maxItems": 50
}
```

### Example: one photo with everything

```json
{
  "mode": "byPhoto",
  "photoIds": ["fWBZ9r4vO9M"],
  "includePhotoDetails": true,
  "includePhotographerStats": true
}
```

### Example: photos related to a seed photo

```json
{
  "mode": "byRelated",
  "photoIds": ["1Z2niiBPg5A"],
  "orientation": "landscape",
  "maxItems": 40
}
```

## Use cases

- **Content and design teams** — source on-brand photography in the right orientation and colour.
- **ML and computer-vision datasets** — build labelled image sets with captions, tags, EXIF, dimensions, and geodata.
- **Photographer research** — track a portfolio's newest work, popular shots, and lifetime reach.
- **Visual discovery** — expand a seed photo into Unsplash's related-photo feed.
- **Collection monitoring** — keep an audit trail of a curated collection or topic over time.
- **Wallpaper and app catalogs** — pull multiple resolutions and BlurHash placeholders in one run.
- **Market and trend analysis** — compare likes, views, and downloads across subjects and visual styles.

## Data source and licence

This actor reads Unsplash's public website data. It is not affiliated with, endorsed by, or sponsored by Unsplash.

Photos are covered by the [Unsplash License](https://unsplash.com/license): free for commercial and non-commercial use, no permission needed, and **attribution is appreciated but not required**. You may not sell unmodified photos or compile Unsplash images to replicate a competing image service. Unsplash+ photos are covered by the separate Unsplash+ License and need an Unsplash+ plan for full-size files; their CDN images are watermarked.

Unsplash's API guidelines ask apps to trigger `links.download_location` when a user downloads a photo. That endpoint requires Unsplash API credentials. This actor is an independent website scraper: it returns `downloadUrl` (a direct CDN file) and `downloadPageUrl` (Unsplash's tracked web download page) instead of calling `download_location`. If you build a product on the official API, follow its guidelines and use your own credentials.

## FAQ

**Do I need an Unsplash API key or account?**
No. All modes work without an account, API key, login, or cookies.

**How many photos can I get, and can I pass the 4,000-per-query search limit?**
`maxItems` caps a run at 1,000 photos. A single keyword search has its own 4,000-row window. To go wider, partition the query: run it separately for landscape / portrait / squarish, for different dominant colours, or for free vs Unsplash+ photos, then combine the rows by `id`. Topic, collection, photographer, and related feeds can be deeper still.

**Why are `views` and `downloads` missing on some rows?**
Unsplash publishes per-photo stats for free photos only. Premium photo stats are suppressed, so the actor omits them instead of returning a misleading zero. Photographer lifetime totals are still available when enabled.

**Why are EXIF, location, or tags missing?**
They are photographer-supplied and only present on some photos. The detail pass adds them when Unsplash has them; otherwise the field is omitted.

**Why is there no family-friendly flag?**
Unsplash's JSON source does not expose it. The flag only appears in the HTML photo page (300-380 KB decompressed, ~70 KB compressed), and a Range request for the JSON-LD window returns HTTP 416, so the actor does not pay an extra full-page read per photo for one low-value boolean.

**Does the actor return illustrations?**
No. This is a photo actor. Unsplash serves illustrations as a separate asset type; mixing them would change the record schema and the supported filters.

**What is the difference between `downloadUrl`, `downloadPageUrl`, and `sourceUrl`? Does the actor download files?**
`downloadUrl` and the `imageUrls` variants are direct CDN files that work from scripts and integrations. `downloadPageUrl` and `sourceUrl` are Unsplash web pages protected by its bot check; they open normally in a browser. The actor returns metadata and URLs, not image binaries — use your own downloader if you need the files.

**Can I use the photos commercially?**
Yes, under the Unsplash License, including commercial use. Attribution is appreciated but not required. Do not sell unmodified photos or build a competing image service from Unsplash content. Check the licence for every use case.

**Why does a portfolio or topic stop before `maxItems`?**
The source may have fewer photos than requested, or a filter (orientation/free-only) removed candidates. The actor only counts photos that pass every filter, so it never returns short by silently including filtered-out rows.

**Do I need to configure a proxy? How fresh is the data?**
No proxy setup is needed; the actor chooses a working connection automatically. Every run reads Unsplash live, with nothing cached between runs.
