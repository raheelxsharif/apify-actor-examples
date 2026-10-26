# iQIYI Album Profile & Channel Ranking Scraper

iQIYI (iq.com) hosts Chinese, Korean, and Thai dramas, movies, variety shows, and anime for an international audience. This actor pulls three different views of that catalog: a full profile for a specific title, the current top-ranked titles for a channel, and the suggestions iQIYI's own search box returns for a query.

## What this actor does

- Looks up one or more albums by slug or URL and returns their full profile: genres, cast, episode count, score, release year, and the countries where the title is licensed.
- Pulls the current top-ranked titles for a channel (Top, Movie, Series, Kids, Variety, or Anime).
- Returns the autocomplete-style title suggestions iQIYI offers for a search term.
- Caps every run at a configurable number of records.
- Fields iQIYI doesn't publish for a given title (a missing score, an untagged genre group) are left out of that record rather than sent empty.

## Output per mode

### `byId` — album profile

- `albumId` — the slug used to look up the title
- `qipuId` — iQIYI's internal content ID
- `title`, `subtitle`, `alterTitle` — title variants
- `description` — synopsis
- `releaseYear`, `releaseStatus` — release year and current airing/release status
- `totalEpisodes`, `durationMinutes` — episode count and per-episode runtime
- `score` — iQIYI user rating
- `channelId` — numeric channel the title belongs to
- `TypeDsj`, `SubjectMatter`, `Language`, `Place`, `Adaptation` — genre/category tag groups (each an array of tag names, present only when iQIYI tags that group)
- `starring`, `creator` — cast and director/creator names
- `posterUrl`, `bannerUrl`, or `landscapeUrl` — one cover image URL, under whichever of these three keys matches the first image iQIYI exposes for the title
- `availableRegions` — ISO country codes where the title is licensed
- `isPremium` — true when the title requires a paid membership
- `userLevel` — minimum membership tier required, when iQIYI publishes one
- `iqiyiUrl`, `sourceUrl` — links back to the album page
- `recordType` — always `album`
- `scrapedAt` — UTC timestamp of the fetch

### `trending` — channel ranking

- `name` — title name
- `qipuId`, `albumId` — identifiers for the ranked title
- `publishDate` — the date iQIYI shows for the title in the ranking
- `posterUrl` — cover image
- `rating` — iQIYI's rating for the title
- `playCount` — view/play count behind the ranking
- `iqiyiUrl`, `sourceUrl` — links back to the album page
- `recordType` — always `album`
- `scrapedAt` — UTC timestamp of the fetch

### `search` — suggestions

- `suggestion` — the suggested title text iQIYI's search returns for the query
- `sourceUrl` — the search URL the suggestion was generated from
- `recordType` — always `suggestion`
- `scrapedAt` — UTC timestamp of the fetch

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `byId` | `byId` (album profile), `trending` (channel ranking), or `search` (title suggestions) |
| `albumIds` | array | `[]` | Album slugs or full iq.com album URLs; used when `mode` is `byId` |
| `channelId` | integer | `2` | Ranking channel: 0 = Top, 1 = Movie, 2 = Series, 6 = Kids, 7 = Variety, 8 = Anime; used when `mode` is `trending` |
| `searchQuery` | string | `love` | Text to get title suggestions for; used when `mode` is `search` |
| `maxItems` | integer | `25` | Maximum number of records to return (1-1000) |

### Example: pull a drama's full profile

```json
{
  "mode": "byId",
  "albumIds": ["genius-girlfriend-2026-28fzrln6il1"]
}
```

### Example: today's top series

```json
{
  "mode": "trending",
  "channelId": 2,
  "maxItems": 30
}
```

### Example: top anime ranking

```json
{
  "mode": "trending",
  "channelId": 8,
  "maxItems": 20
}
```

### Example: title suggestions for a query

```json
{
  "mode": "search",
  "searchQuery": "love",
  "maxItems": 10
}
```

## Use cases

- Build a catalog of iQIYI originals and licensed titles with genre and cast metadata.
- Check which countries a specific drama, movie, or anime is currently licensed in.
- Track which titles are trending on a given channel over time.
- Feed genre tags, cast, and ratings into a recommendation or research dataset.
- Check what iQIYI's own search surfaces for a keyword before building a campaign around it.

## FAQ

**Is this affiliated with iQIYI?**
No. This is an independent actor that reads iQIYI's public pages; it has no relationship with iQIYI or iq.com.

**Do I need an iQIYI account or subscription?**
No. Every field comes from pages and lookups that are open to the public.

**Why does `search` return short suggestions instead of full search results?**
iQIYI's public search box only exposes suggestion-style matches without logging in, so that's what this mode returns rather than a full results page.

**Why is a field like `score` or a genre tag group missing from a record?**
iQIYI doesn't tag or rate every title the same way — a documentary may have no `score`, and a title with no romance elements has no `SubjectMatter` entry for it. Missing fields are left out instead of being sent as empty values.

**Why does the `trending` mode return fewer fields than `byId`?**
The channel ranking view only exposes a summary of each title (name, rating, play count); run `byId` with the returned `albumId` to get the full profile.

**How many records can one run return?**
Up to `maxItems`, capped at 1000 per run.

**How current is the data?**
Every run queries iQIYI live, so rankings and profiles reflect what iq.com shows at run time.
