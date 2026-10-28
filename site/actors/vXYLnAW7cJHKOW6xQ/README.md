# NetEase Cloud Music Song, Chart & Playlist Scraper

NetEase Cloud Music (网易云音乐) is one of China's largest music streaming services. This actor reads its public song, artist, playlist, and chart pages without a login, so you can pull track metadata, artist catalogues, and top-100 charts into a dataset.

## What this actor does

- Searches songs by free-text query and returns matching tracks.
- Fetches full song details — including popularity, play/collection counts, and optional lyrics — for one or more song URLs.
- Lists an artist's top songs or full album discography.
- Browses playlists by genre category, sorted by popularity.
- Pulls any of NetEase's top-100 charts, with an optional chart-summary record before the tracks.
- Filters by song duration, minimum popularity, or a keyword in the title.
- Leaves out fields a song, album, or playlist doesn't have published rather than sending them empty.

## Output per record type

Every data record carries `sourceUrl` and `scrapedAt` (UTC ISO). Records that couldn't be built from a bad input carry `recordType: "error"` with `errorCode`, `errorMessage`, and `inputValue` instead.

### Song (`recordType: "song"`)

- `songId`, `name`, `alias`, `transName`
- `artists`, `artist` — all artist names, and the first one
- `albumId`, `album`, `coverUrl`
- `durationSec`
- `popularity`, `score`, `playedNum`, `starredNum`
- `fee`, `copyright`, `copyrightId`, `single`
- `mvid` — associated music video ID, when one exists
- `disc`, `trackNo`, `position`
- `publishTimeMs`, `commentThreadId`
- `mp3Url` — when NetEase exposes a direct playback URL
- `lyric` — first portion of the LRC lyric text, only when `includeLyric` is on and the mode is `bySongUrl`

### Album (`recordType: "album"`, mode `byArtistAlbums`)

- `albumId`, `name`, `coverUrl`, `trackCount`
- `publishTimeMs`, `company`, `type`, `subType`
- `paid`, `onSale`
- `artists`, `artist`

### Playlist (`recordType: "playlist"`, mode `byPlaylist`)

- `playlistId`, `name`, `coverUrl`
- `playCount`, `trackCount`, `totalDurationMs`
- `creatorName`, `creatorId`, `creatorAvatarUrl`
- `tags`, `description`
- `subscribedCount`, `commentCount`, `shareCount`
- `updateTimeMs`

### Toplist (`recordType: "toplist"`, mode `byToplist`, when `includeToplistInfo` is on)

- `toplistId`, `name`, `coverUrl`, `description`
- `playCount`, `trackCount`, `subscribedCount`, `updateFrequency`

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `search` | `search`, `bySongUrl`, `byArtist`, `byArtistAlbums`, `byPlaylist`, or `byToplist` |
| `searchQuery` | string | `周杰伦` | Free-text query; used by `search` |
| `songUrls` | array | `[]` | `music.163.com` song URLs; used by `bySongUrl` |
| `artistId` | integer | – | NetEase artist ID, e.g. `6452` for Jay Chou; used by `byArtist`/`byArtistAlbums` when `artistName` is empty |
| `artistName` | string | – | Artist name, resolved to an ID automatically when `artistId` is empty |
| `playlistCat` | string | `华语` | Playlist genre category to browse; used by `byPlaylist` |
| `playlistOrder` | string | `hot` | `hot` (most popular) or `new` (newest); used by `byPlaylist` |
| `toplistId` | string | – | Which top-100 chart to fetch; used by `byToplist` |
| `includeToplistInfo` | boolean | `true` | Emit the chart summary record before its tracks |
| `includeLyric` | boolean | `false` | Attach the start of the lyric text to song records; `bySongUrl` only |
| `minDurationSec` | integer | – | Drop songs shorter than this |
| `maxDurationSec` | integer | – | Drop songs longer than this |
| `minPopularity` | integer | – | Drop songs with popularity below this (0-100) |
| `containsKeyword` | string | – | Keep only songs whose title contains this text |
| `maxItems` | integer | `30` | Hard cap on total records returned, including any error records (1-1000) |

### Example: search for an artist's songs

```json
{
  "mode": "search",
  "searchQuery": "周杰伦",
  "maxItems": 30
}
```

### Example: song detail with lyrics

```json
{
  "mode": "bySongUrl",
  "songUrls": ["https://music.163.com/#/song?id=186016"],
  "includeLyric": true
}
```

### Example: an artist's top songs, filtered

```json
{
  "mode": "byArtist",
  "artistName": "周杰伦",
  "minPopularity": 50,
  "maxItems": 20
}
```

### Example: a top-100 chart

```json
{
  "mode": "byToplist",
  "toplistId": "3778678",
  "includeToplistInfo": true,
  "maxItems": 100
}
```

## Use cases

- Track how an artist's catalogue and album list grows over time.
- Pull a genre playlist for a curated listening session or research dataset.
- Monitor a chart's top tracks and their popularity scores.
- Collect song metadata and lyrics for text or music-trend analysis.
- Build a cross-reference between song, album, and artist identifiers.

## FAQ

**Is this affiliated with NetEase?**
No. This actor reads NetEase Cloud Music's own public pages and has no relationship with NetEase.

**Do I need an account to use this?**
No. Every mode works without logging in.

**Why does `playlistOrder: "new"` return no results?**
NetEase's own newest-playlist listing currently returns nothing for every genre category when queried this way; use the default `hot` order until that changes upstream.

**Why do some songs have no `popularity`?**
Search results sometimes omit a popularity score; those songs are dropped when `minPopularity` is set, since a song with an unknown popularity can't be confirmed to meet the floor.

**Can I look up an album directly by ID?**
Not as a single lookup. Album metadata comes from `byArtistAlbums`, and an album's tracks appear through the playlist, artist, or toplist modes.

**Why did I get fewer songs than `maxItems` from `byArtist`?**
NetEase's top-songs listing for an artist returns a bounded set of tracks; this actor deduplicates by song ID and stops once a page adds nothing new, so very large artists may still return fewer than `maxItems`.

**What happens if I pass an invalid song URL or toplist ID?**
That input produces an `error` record (with `errorCode`, `errorMessage`, and `inputValue`) instead of being silently skipped, and it still counts toward `maxItems`.

**How current is the data?**
Each run reads NetEase Cloud Music live, so popularity, play counts, and chart positions reflect what the site shows at run time.
