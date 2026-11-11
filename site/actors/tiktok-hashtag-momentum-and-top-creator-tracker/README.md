# TikTok Hashtag Momentum & Top Creator Tracker

Read TikTok's own ranked trending-hashtag and trending-video charts from the public Creative Center. For every market you choose, this actor returns exact post and view counts, a day-by-day popularity curve, a plain-language momentum read (rising, peaking, fading), the creators TikTok credits for each trend, and — when you ask for videos — the engagement metrics behind what is winning right now. No TikTok login, cookies or proxy setup.

## What this actor does

- **Two collections:** `trendingHashtags` reads TikTok's ranked hashtag charts; `trendingVideos` reads its ranked trending-video charts.
- **27 hashtag markets, 3 windows:** pick any mix of 27 markets and 7/30/90-day windows. The overall chart plus all 19 industry charts are available, widening one market/window from 3 hashtags to up to 60.
- **Momentum, not just a snapshot:** every hashtag carries TikTok's daily popularity curve plus a derived momentum score, trend direction, trend stage, peak date and volatility.
- **Creator leaders:** every trending hashtag includes TikTok's top-3 credited creators; an optional leaderboard aggregates them across the whole run so you can see who keeps appearing.
- **Trending videos:** five markets with public video charts, ranked by views, engagement rate or 6-second view-through rate, with author followers, lifetime metrics and optional top comments. Content-tag rankings (Beauty & Care, Fashion, Food & Beverage, Lifestyle & Leisure, Technology & Finance) can be queried individually or all at once, deduping videos for up to ~24 per market/window/sort instead of 4.
- **Honest caps:** TikTok's public charts show 3 hashtags per ranking and 4 videos per video ranking to guests. The actor explains its coverage in the run status instead of implying more.
- Empty fields are omitted — never returned as `null`, `""` or `[]`.

## Output per hashtag (`rowType: "hashtag"`)

- `hashtagId`, `hashtagName`, `hashtagUrl` — identity and the durable tag page link
- `rankIndex`, `isOverallRanking`, `rankingSlices` — rank, whether it made the overall chart, and which industry charts it also ranked in
- `publishCnt` — posts published with the hashtag during the window
- `vv` — total video views those posts earned during the window
- `avgViewsPerPost` — views divided by posts
- `popularityCurve` — daily `{date, value}` points from TikTok's normalized 0–100 index
- `momentumChangePct`, `trendDirection`, `trendStage`, `peakDate`, `peakValue`, `curveVolatility` — derived momentum signals
- `industryIds`, `industryNames` — TikTok's category tags for the hashtag
- `topCreators[]` — TikTok's credited creators (`rank`, `ttuid`, `handleName`, `nickname`, `followerCount`, `countryCode`, `avatarUrl`)
- `observedCountry`, `observedPeriod`, `scrapedAt`, `dataSource`

## Output per creator (`rowType: "creator"`, optional)

- `creatorId`, `handleName`, `nickname`, `followerCount`, `countryCode`, `avatarUrl`
- `appearances` — how many collected hashtag rows credited this creator
- `hashtags[]`, `bestRank`, `observedCountries[]`, `observedPeriods[]`, `scrapedAt`

## Output per video (`rowType: "video"`)

- `videoId`, `videoUrl`, `embedUrl`, `caption`, `hashtags[]`
- `createTime`, `createdAt`, `ageDays`, `coverUrl`
- `author` — `id`, `username`, `displayName`, `bio`, `followerCount`, `avatarUrl`
- `views`, `organicViews`, `engagementRate`, `sixSecondViewRate` — window metrics
- `viewsLifeTime`, `organicViewsLifeTime`, `engagementRateLifeTime`, `sixSecondViewRateLifeTime` — lifetime metrics
- `viewsPerDay`, `lifetimeViewsShare` — derived velocity
- `topComments[]`, `commentKeywords[]` — only with Include top comments
- `contentTag` — the content-tag ranking that surfaced this video (omitted for the unfiltered ranking)
- `rankingKey` — provenance key: `market:window:sort:weeksBack:tag` (`all` = unfiltered)
- `observedCountry`, `observedPeriod`, `videoSortBy`, `weeksBack`, `scrapedAt`

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | select | `trendingHashtags` | `trendingHashtags` or `trendingVideos` |
| `countries` | multi-select | `["US"]` | 27 markets for the hashtag chart |
| `periods` | multi-select | `["7"]` | Hashtag windows: 7 / 30 / 90 days |
| `includeOverallRanking` | boolean | `true` | Include each market's overall top-3 chart |
| `scanAllIndustries` | boolean | `true` | Read all 19 industry charts (up to 60 hashtags per market/window) |
| `industries` | multi-select | `[]` | Specific industry charts when `scanAllIndustries` is off |
| `includeCreatorLeaders` | boolean | `false` | Emit the aggregated creator leaderboard |
| `minHashtagAppearances` | integer | `2` | Minimum trend appearances for a creator row |
| `videoCountries` | multi-select | `["US"]` | Video chart markets: US, JP, VN, TH, ID |
| `videoPeriods` | multi-select | `["7"]` | Video windows: 7 / 30 days |
| `videoSortBy` | select | `views` | `views`, `engagementRate`, `sixSecondViewRate` |
| `videoOrganicOnly` | boolean | `false` | Rank only organic (non-promoted) videos |
| `videoContentTags` | multi-select | `[]` | Content-tag rankings; each selected tag is a separate ranking read, deduped by video. `all` = unfiltered + all five tags |
| `weeksBack` | integer | `0` | Extra past weekly video rankings (0–4) |
| `includeTopComments` | boolean | `false` | Add top comments and keywords (up to 100 videos) |
| `maxItems` | integer | `1000` | Maximum hashtag/video rows per run |

### Example: this week's US hashtags, every industry

```json
{
  "mode": "trendingHashtags",
  "countries": ["US"],
  "periods": ["7"],
  "scanAllIndustries": true,
  "includeCreatorLeaders": true
}
```

### Example: compare three markets over 30 days

```json
{
  "mode": "trendingHashtags",
  "countries": ["US", "GB", "JP"],
  "periods": ["30"],
  "industries": ["14000000000", "27000000000"],
  "includeOverallRanking": false
}
```

### Example: trending videos in Japan and the US

```json
{
  "mode": "trendingVideos",
  "videoCountries": ["US", "JP"],
  "videoPeriods": ["7"],
  "videoSortBy": "engagementRate",
  "weeksBack": 2,
  "includeTopComments": true
}
```

## Use cases

- **Content calendars** — slot a market's current hashtags, with trend stage, into next week's posting plan.
- **Trend timing** — separate hashtags that are still breaking from ones that already peaked.
- **Creator partnerships** — see which creators TikTok credits across several rising trends before reaching out.
- **Market comparison** — run the same window across markets to find local trends rather than global noise.
- **Brand and campaign monitoring** — schedule a daily run and watch your tag enter, climb or leave a market's chart.
- **Video performance research** — study the engagement rate and 6-second view-through rate of videos TikTok is surfacing right now.

## FAQ

**Why does one market return only a few hashtags?**
TikTok's public Creative Center charts show the top 3 hashtags per ranking to visitors who are not signed in. This actor reads the overall chart plus the 19 industry charts, so a market and window yields up to 60 hashtags in total. There is no public path to a deeper ranking without a TikTok account.

**What does the momentum score mean?**
`momentumChangePct` compares the average of the most recent third of the popularity curve with the average of the third before it. It is a percentage of TikTok's normalized 0–100 index, not raw views. Values of +10% or more are labelled rising, -10% or less falling.

**Why are video charts limited to five markets?**
TikTok publishes trending-video charts publicly for the United States, Japan, Vietnam, Thailand and Indonesia. Hashtag charts cover 27 markets. The input lists exactly the markets that return video data.

**How many videos can one ranking return?**
The public video chart shows 4 videos per market, window and sort. Adding extra weekly windows or selecting several markets or windows widens coverage; rows are deduplicated by video.

**Why are comments optional?**
Top comments are read separately for each video, so they are opt-in and capped at 100 videos per run.

**Are the image links permanent?**
`coverUrl` and creator `avatarUrl` are signed TikTok CDN links that expire after about 48 hours. The durable links are `videoUrl`, `embedUrl` and `hashtagUrl`, which always resolve.

**Why do some hashtags have no industry?**
TikTok assigns industry categories to only some hashtags. When it publishes none, `industryIds` and `industryNames` are omitted for that row.

**Does this actor need cookies or a proxy?**
No. It reads TikTok's public Creative Center data over plain HTTP. There is nothing to log in to and no proxy to configure.

**How fresh is the data?**
The charts refresh continuously. For trend tracking, schedule one run per market per day; repeated runs at the same time return the same ranking.

**Is this affiliated with TikTok?**
No. This is a third-party actor that reads publicly available Creative Center trend data; it is not affiliated with TikTok or ByteDance.
