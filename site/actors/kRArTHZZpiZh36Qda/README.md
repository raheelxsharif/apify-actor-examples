# DeviantArt Artwork Search, Gallery & Trending Feed

DeviantArt is one of the longest-running communities for digital art, traditional art, photography, and fan art. This actor reads its public RSS feeds to return deviations matching a keyword search, an artist's full gallery, or the site's currently trending artwork — with no account or API key needed.

## What this actor does

- Searches DeviantArt for deviations matching a keyword.
- Pulls every deviation in a specific artist's public gallery by username.
- Reads the currently trending/popular deviations feed, optionally scoped to one art category.
- Filters results to a single category — Digital Art, Traditional Art, Photography, Fan Art, Literature, Film & Animation, or Designs.
- Any field a deviation's feed entry doesn't include (such as `category` when the feed omits one) is left out of that record.

## Output per deviation

- `deviationId` — numeric ID parsed from the deviation's URL
- `title` — artwork title
- `artistName` — the creator's username
- `category` — the art category the feed tags the deviation with, when present
- `imageUrl` — preview image URL
- `deviationUrl` — direct link to the deviation's page
- `publishedAt` — publication date as given by the feed
- `recordType` — always `deviation`
- `scrapedAt` — collection timestamp

## Input

| Field | Type | Default | Description |
| --- | --- | --- | --- |
| `mode` | String (select) | `search` | `search`, `gallery`, or `popular`. |
| `searchQuery` | String | `fantasy art` | Keyword to search for, used by `search`. |
| `username` | String | – | DeviantArt username whose gallery to browse, used by `gallery`. |
| `category` | String (select) | – | Restrict to one category: Digital Art, Traditional Art, Photography, Fan Art, Literature, Film & Animation, or Designs. |
| `maxItems` | Integer | 30 | Maximum number of deviation records to return (1-500). |

### Example: keyword search

```json
{
  "mode": "search",
  "searchQuery": "fantasy art",
  "maxItems": 30
}
```

### Example: one artist's gallery

```json
{
  "mode": "gallery",
  "username": "example-artist",
  "maxItems": 50
}
```

### Example: trending photography

```json
{
  "mode": "popular",
  "category": "photography",
  "maxItems": 40
}
```

## Use cases

- **Art discovery** — pull a fresh batch of artwork matching a theme or keyword.
- **Artist portfolio tracking** — monitor what a specific artist has published lately.
- **Trend spotting** — scan the popular feed by category to see what's gaining traction right now.
- **Reference gathering** — collect image URLs and artist credits for mood boards or research.
- **Category comparisons** — pull the trending feed across several categories to compare activity levels.

## FAQ

**Do I need a DeviantArt account or API key?**
No — every mode reads DeviantArt's public RSS feeds, which don't require authentication.

**Why is `category` sometimes missing from a record?**
The feed only includes a category tag for some deviations; when it's absent on the source feed entry, the field is left off that record.

**Does `category` filtering work the same in every mode?**
In `popular` mode, the category is applied as part of the trending feed query itself. In `search` and other filtering, it's matched against whatever category tag the feed entry carries, so results can vary depending on how consistently deviations are tagged.

**Can I combine a search keyword with a category filter?**
`category` filtering is available alongside `search` and `popular`; `gallery` mode returns everything in the artist's feed regardless of category.

**How fresh is the data?**
Each run reads the live RSS feeds at request time, so results reflect what's currently published.

**Why might a gallery return fewer deviations than expected?**
The RSS feed for a gallery reflects what DeviantArt currently exposes publicly for that account; private or mature-filtered content outside the feed's scope won't appear.

**Is this affiliated with DeviantArt?**
No. This actor reads DeviantArt's public RSS feeds and is not affiliated with DeviantArt, Inc.
