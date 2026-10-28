# NetEase Cloud Music Scraper: Songs, Charts & Comments

NetEase Cloud Music (网易云音乐) is one of China's largest music streaming catalogs. This actor reads its public catalog without an account: search songs, albums, artists and playlists; pull song details with full LRC, translated, romanized and word-by-word lyrics; walk an artist's complete discography; read album and playlist tracklists; collect every current chart; and export comments with optional replies. No login, cookies, API keys or proxy setup.

## What this actor does

- **Ten modes:** `search`, `bySongUrl`, `byArtist`, `byArtistAlbums`, `byAlbum`, `byPlaylist`, `byPlaylistUrl`, `byToplist`, `toplists`, `comments`.
- **Catalog search:** songs, albums, artists and playlists through NetEase's modern search endpoint, with popularity and duration included.
- **Lyrics:** full LRC plus translation, romanization and word-by-word lyrics whenever NetEase publishes them.
- **Playlists and charts:** complete tracklists from `trackIds`, including each track's playlist rank and the time it was added.
- **Comments:** recommended or hot comments for songs, albums, playlists and charts, with an optional reply floor.
- **Clean output:** empty fields are omitted; invalid or unavailable inputs become explicit `error` records instead of disappearing.

## Services

- **Music catalog lookup** — song, album, artist and playlist search.
- **Song enrichment** — release dates, quality tiers, maximum bitrate, music-video links and full lyric variants.
- **Artist intelligence** — full song catalogue, album catalogue, profile biography, images, ranks, identities and fan count.
- **Playlist and chart coverage** — 70 live playlist categories, all 65 current charts, chart positions and rank movement where NetEase publishes it.
- **Audience research** — comment text, likes, timestamps, authors, avatars, IP locations and replies.

## How it works

The actor reads NetEase Cloud Music's public web catalog anonymously. Each mode calls the matching public catalog, lyric, chart or comment view and converts the response into flat, readable records. When NetEase responds with a verification challenge instead of data, the actor keeps the run alive by switching to a working connection automatically and continues from the same page. Runs stop on the requested item cap or when NetEase runs out of matching results — never on a fixed time limit.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `search` | One of the ten modes listed above |
| `searchQueries` | array | `["周杰伦"]` | Queries for `search` |
| `searchType` | string | `song` | `song`, `album`, `artist` or `playlist` |
| `songUrls` | array | – | Song URLs or bare IDs for `bySongUrl` |
| `artistIds` | array | – | Artist IDs, URLs or exact names for `byArtist` / `byArtistAlbums` |
| `artistOrder` | string | `hot` | `hot` or `time` for an artist's songs |
| `albumUrls` | array | – | Album URLs or IDs for `byAlbum` |
| `playlistUrls` | array | – | Playlist URLs or IDs for `byPlaylistUrl` |
| `playlistCat` | string | `华语` | 70 NetEase categories plus `全部` for `byPlaylist` |
| `toplistIds` | array | – | Chart IDs, names or URLs for `byToplist`; `all` or empty scans every chart |
| `maxTracksPerChart` | int | `0` | Tracks per chart; `0` reads all |
| `commentUrls` | array | – | Song/album/playlist/chart URLs or IDs for `comments` |
| `commentOrder` | string | `recommended` | `recommended` or `hot` |
| `maxCommentsPerItem` | int | `100` | Top-level comments per resource (1–1,100) |
| `includeReplies` | boolean | `false` | Also read reply floors |
| `maxRepliesPerComment` | int | `20` | Replies read per comment |
| `includeArtistProfile` | boolean | `false` | Emit an artist profile record |
| `includeLyrics` | boolean | `false` | Attach every available lyric variant |
| `includeAlbumTracks` | boolean | `true` | Emit album tracks as songs |
| `includePlaylistTracks` | boolean | `true` | Emit the complete playlist tracklist |
| `includeToplistInfo` | boolean | `true` | Emit a chart summary before tracks |
| `minDurationSec` / `maxDurationSec` | int | – | Song duration window; metadata records are unaffected |
| `minPopularity` | int | – | Song popularity floor (0–100) |
| `minPlayCount` / `minTrackCount` | int | – | Playlist browse filters |
| `containsKeyword` | string | – | Name must contain this text (case-insensitive) |
| `maxItems` | int | `100` | Hard cap on all dataset records, including errors (1–100,000) |

### Example: search songs

```json
{
  "mode": "search",
  "searchQueries": ["周杰伦", "晴天"],
  "searchType": "song",
  "maxItems": 50
}
```

### Example: song details with every lyric variant

```json
{
  "mode": "bySongUrl",
  "songUrls": ["https://music.163.com/#/song?id=186016"],
  "includeLyrics": true,
  "maxItems": 1
}
```

### Example: a complete playlist with added-at timestamps

```json
{
  "mode": "byPlaylistUrl",
  "playlistUrls": ["https://music.163.com/#/playlist?id=8803637201"],
  "includePlaylistTracks": true,
  "maxItems": 500
}
```

### Example: all current charts, first 20 tracks each

```json
{
  "mode": "byToplist",
  "toplistIds": ["all"],
  "maxTracksPerChart": 20,
  "includeToplistInfo": true,
  "maxItems": 1000
}
```

### Example: hot comments with replies

```json
{
  "mode": "comments",
  "commentUrls": ["https://music.163.com/#/song?id=186016"],
  "commentOrder": "hot",
  "maxCommentsPerItem": 100,
  "includeReplies": true,
  "maxRepliesPerComment": 10,
  "maxItems": 200
}
```

## Output

Every record carries `recordType`, `sourceUrl` and `scrapedAt` (UTC ISO). Empty fields are omitted at every depth.

### Song (`recordType: "song"`)

- `songId`, `name`, `alias[]`, `transName`
- `artists[]`, `artist`, `albumId`, `album`, `coverUrl`
- `durationSec`, `popularity`, `score`, `playedNum`, `starredNum`
- `fee`, `copyright`, `copyrightId`, `single`
- `mvid`, `mvUrl`, `disc`, `trackNo`, `position`
- `publishTimeMs`, `publishDate`, `commentThreadId`
- `availableQualities[]`, `maxBitrateKbps`
- `lyric`, `lyricTranslation`, `lyricRomanization`, `lyricWordByWord`, `lyricCredit`
- `rankPosition`, `chartId`, `chartName`, `previousRank`, `rankChange`
- `playlistId`, `playlistName`, `addedToPlaylistAt`
- `mp3Url` only when NetEase exposes a valid HTTPS source value

### Album (`recordType: "album"`)

`albumId`, `name`, `coverUrl`, `trackCount`, `publishTimeMs`, `publishDate`, `company`, `type`, `subType`, `paid`, `onSale`, `artists[]`, `artist`, `description`, `commentThreadId`, `albumUrl`, `sourceUrl`.

### Artist (`recordType: "artist"`)

`artistId`, `name`, `alias[]`, `transNames[]`, `briefDesc`, `avatarUrl`, `coverUrl`, `picUrl`, `albumSize`, `musicSize`, `mvSize`, `videoCount`, `fansCount`, `fansSize`, `rank`, `identities[]`, `sourceUrl`.

### Playlist (`recordType: "playlist"`)

`playlistId`, `name`, `coverUrl`, `playCount`, `trackCount`, `totalDurationMs`, `creatorName`, `creatorId`, `creatorAvatarUrl`, `tags[]`, `description`, `updateTimeMs`, `createTimeMs`, `commentThreadId`, `playlistType`, `privacy`, `updateFrequency`, `sourceUrl`.

### Chart summary (`recordType: "toplist"`)

`toplistId`, `name`, `englishTitle`, `coverUrl`, `description`, `playCount`, `trackCount`, `subscribedCount`, `updateFrequency`, `commentThreadId`, `createTimeMs`, `publishDate`, `specialType`, `playlistType`, `sourceUrl`.

### Comment (`recordType: "comment"`)

`commentId`, `parentCommentId`, `content`, `likedCount`, `createdAt`, `createdAtMs`, `authorNickname`, `authorUserId`, `authorAvatarUrl`, `ipLocation`, `isHot`, `replyCount`, `repliedToCommentId`, `repliedToUser`, `repliedToContent`, `threadId`, `resourceType`, `resourceId`, `sourceUrl`.

### Error (`recordType: "error"`)

`errorCode`, `errorMessage`, `inputValue` — emitted for an input that could not be parsed or found. Error records count toward `maxItems`, so a partially valid list still returns every usable record alongside clear failures.

## Use cases

- Build a searchable NetEase catalog slice for a genre, artist or mood.
- Export synchronized, translated and romanized lyrics for language or music research.
- Track chart positions and rank movement over time.
- Audit a playlist's full running order and when each track was added.
- Study listener sentiment and fan culture through comments and replies.
- Mirror an artist's complete discography and album metadata into a local database.

## Usage cost

NetEase access is free and needs no subscription. You pay for the Apify run itself and the actor's Store price, which is shown on the listing. A small search run costs a fraction of a cent; large playlist, chart or comment runs cost more because they process more records. The daily prefill is deliberately tiny (`search`, one query, three records).

## FAQ

**Do I need a NetEase account, cookie or API key?**
No. Every mode reads public data anonymously. Nothing in the input asks for credentials.

**Do I need to configure a proxy?**
No. The actor chooses a working connection automatically. Just fill in the input and run it.

**Why do comments stop before the total shown on NetEase?**
NetEase's public comment pages stop around 1,100 comments per resource. The actor reads up to 1,100 top-level comments, then stops with a note instead of pretending the rest are available.

**Why do only a few songs show `previousRank` and `rankChange`?**
NetEase publishes rank movement for the first six preview tracks of each chart. Every track still gets its current `rankPosition`; the comparison fields are omitted when the source has no previous rank.

**Why are translation, romanization or word-by-word lyrics missing on some songs?**
NetEase provides those variants only for some tracks. The main LRC is usually present; the other variants are added only when NetEase publishes them.

**Can I get an artist's entire catalog, not just top songs?**
Yes. `byArtist` paginates the full song catalog, and `byArtistAlbums` walks every album. Set `includeArtistProfile` to add a profile record.

**Can I search in Chinese and English?**
Yes. Search queries, artist names and chart names accept Chinese, English and mixed text.

**What happened to the old `playlistOrder: "new"` option?**
NetEase no longer serves a newest-first playlist listing; it returns nothing for every category. The setting is accepted for old saved tasks but is ignored, and the run says so in its status message.

**Is the data live?**
Yes. Each run reads NetEase at run time, so chart positions, play counts and comments reflect the moment of the run.

## Notes

- Bare numeric IDs are accepted alongside music.163.com URLs.
- For comments, a bare ID is treated as a song; use `album:123`, `playlist:123` or `chart:123` when the ID belongs to another resource.
- `toplistIds: ["all"]` or an empty list scans every chart currently published by NetEase.
- Legacy saved inputs (`searchQuery`, `artistId`, `artistName`, `toplistId`, `includeLyric`, `playlistOrder`) continue to work without being shown in the input form.
- API callers can pass a `modes` array (for example `["bySongUrl","byArtist"]`) to run several modes in one run; the console's single `mode` field remains the normal path.
- The chart catalogue is served live when available and falls back to the built-in 65-chart list if NetEase withholds it.
- Song records include a valid `mp3Url` only when NetEase exposes one; the actor does not download audio.

## Limitations

- Comment pagination is capped around 1,100 per resource by NetEase; time-ordered ("newest") comments are not exposed by the public API.
- `previousRank` and `rankChange` exist only for the top six preview tracks of a chart.
- The newest-first playlist browse sort is no longer available upstream.
- Translation, romanization and word-by-word lyrics are conditional on NetEase publishing them.
- Playback/audio URLs are signed and can expire, so audio download is out of scope.
- A temporary NetEase verification challenge can add latency while the actor switches connections; it does not produce an empty run.
