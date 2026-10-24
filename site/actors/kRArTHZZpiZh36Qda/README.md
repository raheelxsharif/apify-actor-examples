# DeviantArt Artwork Search, Gallery & Trending Feed

Collect artwork from DeviantArt through its public pages. Search by keyword, browse a tag, read an artist's gallery, follow the Daily Deviations feed, or open specific artwork URLs — with no account, API key or proxy setup.

## What you get

- **Five input paths**: keyword search, tag browse, artist gallery, Daily Deviations, and direct artwork URLs or IDs.
- **Artwork metadata**: title, artist, publication time, media type, file type, language, licence, and daily-deviation flag.
- **Engagement stats**: views, favourites, comments, downloads and private collections.
- **Rights and provenance flags**: mature, sensitive, downloadable, AI-generated and AI-use-disallowed.
- **Signed media URLs**: thumbnail and full preview links built from the page's current image tokens, plus display dimensions.
- **Optional artwork details**: tags, description, original dimensions, file size, original file type and watermark flag.
- **Optional artist profiles**: watchers, deviation count, pageviews and join date.
- **Literature support**: list modes return a text excerpt for literature submissions, with image fields omitted; detail enrichment merges the page description when present.
- **Omit-empty output**: fields the source does not expose are left out instead of filled with guesses.

## Input

| Field | Type | Default | Used by |
| --- | --- | --- | --- |
| `mode` | Select | `search` | All modes: `search`, `tag`, `gallery`, `daily`, `artwork`. |
| `searchQuery` | Text | `fantasy art` | `search`; keyword or search URL. |
| `tag` | Text | – | `tag`; accepts `fantasy` or a tag URL. |
| `username` | Text | – | `gallery`; accepts a username or profile URL. |
| `artworkUrls` | String list | – | `artwork`; URL, `/art/...-id` link or numeric ID. |
| `contentType` | Select | `all` | `search`; limit to `image` or `literature`. |
| `order` | Select | `recommended` | `tag`; `recommended` or `most-recent`. |
| `includeDetails` | Boolean | `false` | Browse modes only: adds tags, description and original-file metadata with one extra artwork page per record. |
| `includeArtistStats` | Boolean | `false` | Adds profile statistics; one extra page per unique artist. |
| `excludeMature` | Boolean | `false` | Skips artwork flagged mature or sensitive. |
| `maxItems` | Integer | `30` | Stops after this many unique artworks (1–500). |

Every record is de-duplicated by deviation ID across pages.

### Examples

Search for recent fantasy artwork:

```json
{
  "mode": "search",
  "searchQuery": "fantasy art",
  "contentType": "image",
  "maxItems": 50
}
```

Browse a tag with detail enrichment:

```json
{
  "mode": "tag",
  "tag": "fantasy",
  "order": "most-recent",
  "includeDetails": true,
  "maxItems": 20
}
```

Read an artist gallery with profile stats:

```json
{
  "mode": "gallery",
  "username": "example-artist",
  "includeArtistStats": true,
  "maxItems": 40
}
```

Open specific artworks:

```json
{
  "mode": "artwork",
  "artworkUrls": [
    "https://www.deviantart.com/byzwa-dher/art/Temple-485251112",
    "485251112"
  ]
}
```

## Output fields

`deviationId`, `title`, `artistName`, `artistUsername`, `artistUrl`, `artistAvatarUrl`, `artistType`, `deviationUrl`, `shortUrl`, `sourceFeed`, `searchInput`, `recordType`, `scrapedAt`, `publishedAt`, `type`, `mediaType`, `fileType`, `language`, `isMature`, `isNsfg`, `isDownloadable`, `isAiGenerated`, `isAiUseDisallowed`, `isDailyDeviation`, `license`, `imageUrl`, `thumbnailUrl`, `imageWidth`, `imageHeight`, `thumbnailWidth`, `thumbnailHeight`, `views`, `favourites`, `comments`, `downloads`, `privatelyCollected`, `description`.

Detail enrichment (automatic in `artwork` mode, opt-in elsewhere) adds `tags`, `originalWidth`, `originalHeight`, `fileSizeBytes`, `originalFileType` and `hasWatermark`. Artist enrichment adds `artistWatchers`, `artistDeviationsCount`, `artistPageviews` and `artistJoinDate`.

## Use cases

- Find new artwork around a theme, fandom or tag.
- Monitor an artist's public gallery over time.
- Track which genres and creators appear in the Daily Deviations feed.
- Build mood boards or training sets from signed preview images and attribution metadata.
- Compare engagement across tags and creators.

## FAQ

**Do I need a DeviantArt account or API key?**
No. The actor reads public artwork pages and requires no login or developer credentials.

**Why are some tags or descriptions missing?**
In browse modes, detail data is merged when you enable `includeDetails`; `artwork` mode reads it from the same page. DeviantArt does not expose it on every artwork page, so missing values are omitted rather than invented.

**Do image links expire?**
Image links carry signed tokens from the page you scraped. They usually remain valid for a while, but treat them as temporary and download images promptly if you need long-term storage.

**Why did the run return fewer records than requested?**
The source limits how deep public browsing goes, and pagination can end early; gallery pages can stop at the first server-rendered batch. The run reports how many records it actually collected.

**Why is there no category field?**
DeviantArt's public artwork data no longer exposes the old art-category taxonomy. The actor does not guess one from the media type.

**Can I collect mature artwork?**
Anonymous browsing hides most mature work. `excludeMature` lets you filter out anything flagged mature or sensitive.

**Is literature supported?**
Yes. List modes include a text excerpt and omit image-specific fields. When detail enrichment is on, structured rich-text descriptions are converted to readable text; a literature excerpt is always kept.

**How fresh is the data?**
Each run reads the current public pages at request time.

**Is this affiliated with DeviantArt?**
No. This actor is independent and is not affiliated with DeviantArt, Inc.
