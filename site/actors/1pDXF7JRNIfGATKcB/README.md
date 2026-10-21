# Snapchat Topic Spotlight Feed & Content Tag Extractor

Collect public Snapchat Spotlight videos from topic pages, hashtag feeds, keyword search and the anonymous Spotlight feed. Every video row carries its creator, engagement counts, duration, captions, Snapchat's AI title/description/keywords, hashtags, detected language and, when Snapchat provides them, S2I content tags.

Snapchat spreads its public data across several server-rendered pages. This actor reads the surfaces that are actually live today and combines them:

- **Topic pages** — long-tail topic hubs such as `snapchat.com/topic/cappuccino-foam-technique`, with 19-30 videos and a ranked list of related topics.
- **Hashtag pages** — `snapchat.com/tag/food`, with up to 32 Spotlight cards.
- **Explore search** — `snapchat.com/explore/<query>`, with 1-10 topic entities and 11-24 Spotlight cards (including unwatermarked video URLs).
- **Spotlight feed** — `snapchat.com/spotlight`, with the current anonymous feed of about 25 videos.

There is no public pagination on these surfaces. The actor combines a topic's topic/tag page with its Explore results and de-duplicates by snap ID, which usually produces **40-54 unique videos per topic** instead of 19-32.

## Modes

### Topic or tag videos

Paste topic names, hashtags or Snapchat URLs. The actor tries the topic page and the hashtag page, fetches the matching Explore page, and merges the results. Examples:

- `food`
- `#travel`
- `https://www.snapchat.com/tag/food`
- `https://www.snapchat.com/topic/cappuccino-foam-technique`
- `https://www.snapchat.com/explore/viral`

Old `/topic/food` URLs and bare short names keep working: Snapchat retired the generic `/topic/<short>` pages, so the actor automatically falls back to `/tag/food` and Explore.

### Find topics by keyword

Give one or more keywords and get topic rows with the topic slug, display name, Snapchat's reported snap count, thumbnail, icon and rank. Enable **Expand related topics** to add one hop of related topics from each discovered topic's page links.

### Spotlight trending feed

Fetch the current anonymous Spotlight feed. Snapchat serves about 24-26 videos per request and the order can change between requests; there is no cursor to page further.

## Output

One row per video, or one row per topic in topic-search mode.

Video fields:

- `topicId`, `topicDisplayName`, `topicUrl`, `topicSource`, `rank` — where the video came from and its position on that surface
- `snapId`, `storyId`, `compositeStoryId`, `snapUrl`, `embedUrl` — Snapchat identifiers and links
- `videoUrl`, `videoUrlUnwatermarked`, `videoFormat`, `thumbnailUrl` — direct CDN media URLs
- `creatorUsername`, `creatorDisplayName`, `creatorProfileUrl` — creator identity
- `creatorSubscriberCount` — profile subscriber count, only with `enrichVideos`; cached once per creator and omitted when the profile does not expose it (distinct from `creatorFollowerCount`)
- `viewCount`, `likeCount`, `shareCount`, `commentCount`, `recommendCount`, `pendingCommentCount`, `remixCount` — public engagement counts
- `durationSeconds`, `width`, `height` — media properties
- `description`, `caption` — original Snapchat text
- `llmTitle`, `llmDescription`, `llmKeywords` — Snapchat's own AI metadata
- `hashtags`, `detectedLanguage`, `relatedTags` — hashtags, language and S2I content tags
- `soundTitle`, `soundArtist` — sound information when the feed/detail surface provides it
- `uploadedAt`, `scrapedAt` — ISO 8601 UTC timestamps

Topic fields:

- `topicId`, `topicDisplayName`, `topicUrl`, `topicSource`, `rank`
- `numSnaps`, `topicType`, `thumbnailUrl`, `iconUrl`
- `searchQuery`, `relatedTo`
- `scrapedAt`

Fields a surface does not provide are omitted from that row, never written as `null`. `creatorFollowerCount` and `remainingSpotlightCount` remain in the dataset schema for backward compatibility but are not populated because Snapchat no longer exposes them on these pages.

## Input

| Field | Default | Description |
| --- | --- | --- |
| `mode` | `topicFeed` | `topicFeed`, `topicSearch` or `spotlightFeed`. |
| `topics` | `[]` | Topic names, hashtags or URLs for topic-feed mode. |
| `maxResultsPerTopic` | `50` | Maximum videos per topic after merging surfaces. |
| `includeRelatedTags` | `true` | Include S2I content tags when the page provides them. |
| `searchQueries` | `[]` | Keywords for topic-search mode. |
| `maxTopicsPerQuery` | `50` | Maximum topic rows per query, including related topics. |
| `includeRelatedTopics` | `false` | Add one hop of related topics. |
| `enrichVideos` | `false` | Visit each video page for missing AI title, counts, language, S2I tags and sound, plus each creator profile once for `creatorSubscriberCount`. Cached, bounded, and distinct from followers. |
| `maxItems` | `1000` | Maximum rows across the whole run. |

### Example: three topics

```json
{
  "mode": "topicFeed",
  "topics": ["food", "travel", "#gaming"],
  "maxResultsPerTopic": 50,
  "includeRelatedTags": true
}
```

### Example: discover coffee topics

```json
{
  "mode": "topicSearch",
  "searchQueries": ["coffee", "latte art"],
  "maxTopicsPerQuery": 50,
  "includeRelatedTopics": true
}
```

### Example: current Spotlight feed

```json
{
  "mode": "spotlightFeed",
  "maxItems": 25
}
```

### Example: enrich a few videos

```json
{
  "mode": "topicFeed",
  "topics": ["cappuccino-foam-technique"],
  "maxResultsPerTopic": 5,
  "enrichVideos": true
}
```

## Use cases

- **Trend research** — see which Spotlight videos Snapchat surfaces for a topic today and how they perform.
- **Content tagging** — collect Snapchat's AI keywords, hashtags, detected language and S2I tags for a niche.
- **Creator discovery** — find creators with strong engagement inside a topic.
- **Topic mapping** — start from a keyword, discover topic entities, then expand one hop through related topics.
- **Performance tracking** — schedule repeated runs and compare view, like, share, comment and recommend counts over time.

## FAQ

**Do I need a Snapchat account or cookies?**
No. Every surface used here is public and works without a login, cookie or proxy.

**Why do some tags return no results?**
Snapchat serves 19-32 videos on a topic or tag page and 11-24 on its Explore page, and the actor merges both. Some slugs in Snapchat's tag sitemap are stale and return 404 on `/tag/`; the actor retries once, checks the topic page, then uses Explore only when its cards are relevant (an exact topic entity or at least 15% text match). Otherwise the run reports an honest zero-row status. Long non-Latin queries (roughly over 50 percent-encoded bytes) can also return 403 from Snapchat's edge; use a shorter keyword.

**Why is there no next-page option?**
The public web pages do not expose working pagination. `page`, `cursor`, `offset` and `limit` parameters are ignored by Snapchat, so a run reports honestly when a source is exhausted. For more coverage, add related topics or run topic search.

**What are S2I content tags?**
Snapchat's scene-to-interest tags classify what is in a video. They are available on only part of the catalogue, so the `relatedTags` field is omitted when the page does not include them. AI keywords (`llmKeywords`) have much better coverage.

**Why do `creatorFollowerCount` and `remainingSpotlightCount` not appear?**
Snapchat reports follower count as `0` on these public surfaces and no longer serves `remainingSpotlightIds` on topic pages. The fields stay in the schema for old integrations but are not fabricated. With `enrichVideos`, `creatorSubscriberCount` adds the real profile subscriber count when available (up to 200 unique creators per run); it is not the same metric as follower count.

**Why do the video URLs contain long parameters?**
`mo` and `uc` are optional CDN media/context parameters, not signatures or expiry tokens. The bare CDN path also returns the same file: in a 30-URL sample, both the full URL and the URL with the query removed returned 200 `video/mp4`. The actor therefore does not re-host media. Snapchat can still remove or rotate content, so archive the files if you need permanent copies.

**Can I get the trending feed for a specific country?**
No. Snapchat's web feed has no working country parameter; `country`, `geo`, `region`, `countryCode`, locale and country headers all leave the same randomized 24-26 video slice. Use a topic or keyword for targeting instead.

**Can I browse all 643,000 topics from the sitemap?**
No. The topic sitemap is about 97 MB across 65 unordered shards with no popularity ranking. Keyword search plus one-hop related topics is the useful discovery path.

**What does `enrichVideos` add?**
Each video's Spotlight page is read separately, filling missing AI title/description, comment and recommend counts, language, S2I tags and sound. It also fetches each unique creator's profile once (cached, up to 200 per run) for `creatorSubscriberCount`. That number is a profile metric, not a follower count. Failed detail or profile requests keep the original row unchanged.

**Is this affiliated with Snapchat?**
No. This actor reads public Snapchat pages and is not affiliated with Snap Inc.

## Data source and limitations

The actor reads `snapchat.com` server-rendered pages with plain HTTP requests. No browser is required. It does not log in, solve CAPTCHAs, or bypass access controls. Snapchat changes its pages from time to time; the actor falls back between topic, tag and Explore surfaces and reports source exhaustion in the run status.

Known limits: no surface exposes working pagination, so one topic yields 40-56 unique videos from the merged surfaces; stale tag sitemap slugs return 404; long non-Latin Explore queries can return 403; S2I tags are present on only part of the catalogue; follower counts are always reported as zero on public video pages; and the anonymous Spotlight feed cannot be pinned to a country. Comment text and transcripts are out of scope here; they are covered by the Snapchat comments and video-downloader actors.
