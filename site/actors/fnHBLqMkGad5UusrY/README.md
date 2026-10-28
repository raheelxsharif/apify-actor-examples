# Twitter (X) Media Downloader

Download videos, photos, GIFs, audio tracks and video poster images from public **Twitter / X** posts — **no login, no cookies, no browser**. Paste post links or usernames and get every file stored with a public signed download link that works in any browser or `curl`.

## What this actor does

- **Two discovery paths:** direct post links/IDs, or scan usernames/profile links for their latest media posts
- **All media types:** videos, photos, GIFs and video poster images
- **Quality control:** original or compressed photos; any video quality (`2160p` down to `480p`, best or smallest) — or every available quality at once as separate files
- **Audio only:** extract the soundtrack of videos as M4A or MP3
- **Extras:** media from quoted posts, image alt text, post metadata with a derived engagement rate
- **Empty fields are omitted**

## Output per media file

Each row has a `type`: `media file` for a downloaded file or `status` for a run message.

| Field | Description |
|---|---|
| `tweetId`, `tweetUrl` | Post ID and canonical post URL |
| `mediaType` | `video`, `photo` or `gif` |
| `mediaUrl` | Media URL with the selected photo quality applied (videos/GIFs use the chosen quality) |
| `mediaUrlOriginal` | Untouched X CDN URL, without the quality parameter |
| `thumbnailUrl` | Preview image URL for videos and GIFs |
| `posterImageUrl` | Poster image URL on video/GIF rows when poster saving is on |
| `filename` | File name in the key-value store |
| `fileSizeBytes` | Real size of the downloaded file |
| `width`, `height` | Media dimensions in pixels |
| `durationSeconds` | Video or GIF duration |
| `downloadUrl` | Public signed download link for the stored file |
| `downloadStatus` | `finished`, `failed` or `notDownloaded` |
| `uploaderUsername`, `uploaderFollowers` | Handle and follower count of the account that posted the media |
| `tweetText` | Text of the post |
| `likeCount`, `retweetCount`, `replyCount`, `viewCount` | Public engagement counts |
| `engagementRate` | Derived engagement in percent (likes + reposts + replies ÷ followers) |
| `variantQuality`, `variantBitrate` | Quality label (for example `1080p`) and bitrate of the downloaded variant |
| `availableQualities` | Every quality X offers for the video, best first (label, URL, dimensions, bitrate) |
| `qualityCount` | How many video qualities are available |
| `isBestQuality` | Whether the downloaded variant is the best one X offers |
| `mediaSource` | `post`, `quoted post` or `poster image` |
| `altText` | Image description X stores for the photo or video thumbnail |
| `mediaKey`, `focusRects` | Media identifier and X's crop focus rectangles |
| `allowDownload` | Whether X allows downloads for this media |
| `audioFormat` | `m4a` or `mp3` on audio-only rows |
| `message` | Explanation on failed or skipped files |
| `collectedAt` | UTC timestamp |

Status rows carry `status` (`noInput`, `invalidLink`, `invalidUsername`, `notFound`, `rateLimited`, `error`, `noMatchingMedia`, `noRecords`), a human-friendly `message`, and the related `tweetUrl`, `tweetId` or `username`.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `postUrls` | array | – | Post links, `i/web/status` links or bare post IDs |
| `tweetUrls` | array | – | Alias of post links |
| `usernames` | array | – | Profiles to scan for recent media (`@` optional, profile links accepted) |
| `profileUrls` | array | – | Alias of usernames to scan |
| `maxPostsPerUser` | integer | `20` | How many recent posts to check per username (1–100) |
| `mediaTypes` | array | `["video", "photo", "gif"]` | `video`, `photo`, `gif` |
| `imageQuality` | string | `orig` | `orig`, `large`, `medium`, `small` |
| `videoQuality` | string | `best` | `best`, `2160p`, `1080p`, `720p`, `480p`, `worst` — the closest available quality is used |
| `downloadAllQualities` | boolean | `false` | Download every progressive video quality as a separate row and file |
| `includePosterImage` | boolean | `true` | Also save the video/GIF poster image as a photo |
| `includeThumbnails` | boolean | `false` | Alias of poster images |
| `includeAltText` | boolean | `true` | Attach X's image description to the row |
| `audioOnly` | boolean | `false` | Save only the audio track of videos as M4A/MP3; photos and GIFs are skipped |
| `audioFormat` | string | `m4a` | `m4a` or `mp3` when audio-only is on |
| `includeQuotedMedia` | boolean | `true` | Also download media from a post quoted inside the linked post |
| `includeMetadata` | boolean | `true` | Attach post text and engagement counts to every file |
| `saveToFile` | boolean | `true` | Store files and return public signed links; turn off to collect metadata only |
| `proxyConfiguration` | object | – | Optional, for large or repeated runs |

At least one post link or username is required. Post links and usernames can be combined in one run.

### Example: direct post links

```json
{
  "postUrls": [
    "https://x.com/NASA/status/2107095768821576019",
    "2107516524965794217"
  ],
  "mediaTypes": ["video", "photo"],
  "videoQuality": "1080p"
}
```

### Example: scan profiles for recent media

```json
{
  "usernames": ["NASA", "https://x.com/elonmusk"],
  "maxPostsPerUser": 30,
  "mediaTypes": ["video"],
  "includeMetadata": true
}
```

### Example: audio only (MP3)

```json
{
  "postUrls": ["https://x.com/NASA/status/2107095768821576019"],
  "audioOnly": true,
  "audioFormat": "mp3"
}
```

### Example: every video quality

```json
{
  "postUrls": ["https://x.com/NASA/status/2107095768821576019"],
  "mediaTypes": ["video"],
  "downloadAllQualities": true
}
```

### Example record

```json
{
  "type": "media file",
  "tweetId": "2107095768821576019",
  "tweetUrl": "https://x.com/i/status/2107095768821576019",
  "mediaType": "video",
  "mediaUrl": "https://video.twimg.com/amplify_video/.../avc1/1920x1080/video.mp4?tag=16",
  "mediaUrlOriginal": "https://video.twimg.com/amplify_video/.../video.mp4",
  "thumbnailUrl": "https://pbs.twimg.com/amplify_video_thumb/.../img/thumb.jpg",
  "posterImageUrl": "https://pbs.twimg.com/amplify_video_thumb/.../img/thumb.jpg?format=jpg&name=orig",
  "filename": "2107095768821576019_video_1.mp4",
  "fileSizeBytes": 3178943,
  "width": 1920,
  "height": 1080,
  "durationSeconds": 13.3,
  "downloadUrl": "https://api.apify.com/v2/key-value-stores/{STORE_ID}/records/2107095768821576019_video_1.mp4?signature=...",
  "downloadStatus": "finished",
  "uploaderUsername": "NASA",
  "tweetText": "It's World Space Week! ...",
  "likeCount": 7284,
  "retweetCount": 1138,
  "viewCount": 2838162,
  "variantQuality": "1080p",
  "qualityCount": 3,
  "isBestQuality": true,
  "mediaSource": "post",
  "collectedAt": "2026-10-06T18:53:49+00:00"
}
```

## Use cases

- **Content archival** — back up media from accounts you care about
- **Repurposing** — pull video and photo assets for blogs, presentations or social posts
- **Compliance and evidence** — preserve public posts for legal or regulatory records
- **Media research** — build labelled datasets of public video/photo content
- **Audio workflows** — turn public videos into podcast-ready M4A/MP3 tracks
- **Social listening** — enrich posts with engagement metrics for analysis

## FAQ

**Do I need a Twitter/X account or cookies?** No. The actor is fully cookieless and reads public posts and public profile timelines only.

**What media types are supported?** Videos, photos and GIFs. If a post has 3 photos and 1 video, you get all 4 files (plus the video poster when poster saving is on).

**How are video qualities handled?** The actor picks the closest available quality to your preference (`best` and `worst` are also supported). Portrait videos are compared by their real quality — a 1080×1920 video is treated as 1080p. When `downloadAllQualities` is on, every quality becomes its own file.

**What is `posterImageUrl`?** When poster saving is on, every video/GIF row carries the poster image URL at your chosen photo quality, and the poster is also saved as a separate photo row. Turn the option off to keep video/GIF rows only.

**Does it download media from quoted posts?** Yes — by default media inside a quoted post is collected too and marked with `mediaSource: "quoted post"`. Set `includeQuotedMedia` to `false` to keep only the linked post's own media.

**How does audio-only work?** Videos are downloaded and their audio track is extracted as M4A (AAC) or MP3. Photos and GIFs are skipped in this mode. Audio-only requires file saving, so `saveToFile` is enabled automatically.

**What happens to posts with no media or filtered-out media?** The actor emits a `status` row explaining why, so a run is never silently empty.

**Why are download links signed?** Apify key-value store records need a signature for public access; the actor appends it automatically. Paste the URL into any browser or `curl`.

**How long do download links last?** They follow your Apify key-value store retention (typically 7 days on the free tier). Re-run the actor or copy the files to your own storage for permanent archival.

**Can I collect metadata without downloading files?** Yes — set `saveToFile` to `false`. Rows still include the original CDN URLs.

**Can I download from private or protected accounts?** No. The actor only accesses public content, by design.

## Limits

- Public posts only — deleted posts and protected accounts cannot be retrieved.
- Profile scanning is limited to the posts X exposes to logged-out visitors (roughly the newest posts of the account).
- Very large scans may be rate-limited by X; reduce `maxPostsPerUser` or configure a proxy.
- When X marks a media item as not downloadable, the row reports `downloadStatus: "notDownloaded"` with an explanation instead of a file.

## Integrations

Run it on a schedule and push new files to Google Drive, Dropbox, S3 or any webhook through Apify's built-in integrations, or call the actor from the Apify API to feed media pipelines.
