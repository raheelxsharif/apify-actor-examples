# Steam Game Specs, Pricing & Media Extractor

Every Steam store page packs pricing, system requirements, review scores, media and DLC data into one listing. This actor reads that page (plus Steam's underlying game-details data) for any game you name or link to, and returns it as a single structured record per title.

## What this actor does

- Resolves a game by its Steam store URL or by searching its name and taking Steam's own top match.
- Collects pricing, genres, tags, platforms and system requirements for each title.
- Pulls screenshots, trailer links, supported languages, and review statistics.
- Resolves the recommended "More like this" titles into their own app IDs and names.
- Deduplicates by Steam App ID when the same game is reachable through both a URL and a name in one run.
- Every field is always present in the record; values that don't apply to a game come back as `0`, `""`, `false`, or `[]` instead of being left out.

## Output per game

| Field | Description |
| --- | --- |
| `appId` / `url` | Steam App ID and the game's store page URL. |
| `title` | Game title. |
| `headerImage` / `capsuleImage` / `backgroundImage` | Store artwork URLs. |
| `shortDescription` / `detailedDescription` / `aboutTheGame` | Marketing description text at three lengths. |
| `releaseDate` / `comingSoon` | Release date text and whether the game is not yet released. |
| `requiredAge` | Minimum age rating Steam enforces for the store page. |
| `isFree` | Whether the game is free-to-play. |
| `currency` / `priceInitial` / `priceFinal` / `discountPercent` / `priceFormatted` | Pricing in the store's reported currency, before and after any discount. |
| `developers` / `publishers` / `website` | Credits and the game's official website. |
| `genres` / `categories` / `tags` | Steam's genre labels, feature categories (e.g. "Single-player"), and user-defined tags. |
| `platforms` | Supported platforms (windows, mac, linux). |
| `windowsRequirements` / `macRequirements` / `linuxRequirements` | `{minimum, recommended}` system requirement text per platform. |
| `screenshots` | Full-size screenshot URLs. |
| `trailers` | List of `{name, videoUrl, thumbnail}` for each trailer. |
| `supportedLanguages` / `fullyAudioLanguages` | Text-supported languages and the subset with full audio. |
| `totalReviews` / `positiveReviews` / `negativeReviews` | Review counts. |
| `reviewScore` / `reviewScorePercent` | Steam's review label (e.g. "Very Positive") and its underlying 1–9 scale score. |
| `metacriticScore` / `metacriticUrl` | Metacritic score and review link, when Steam links one. |
| `totalAchievements` | Number of Steam achievements the game has. |
| `dlcCount` / `dlcAppIds` | Number of DLC items and their App IDs. |
| `contentDescriptors` | Mature-content warning labels. |
| `legalNotice` | Legal notice text shown on the store page. |
| `similarGames` | List of `{appId, title}` for Steam's "More like this" recommendations. |
| `awards` | List of `{name, year}` for Steam Awards the game has won. |
| `scrapedAt` | ISO timestamp of when the record was captured. |

## Input

| Field | Type | Default | Description |
| --- | --- | --- | --- |
| `startUrls` | Array of URLs | – | Steam store page URLs to scrape, e.g. `https://store.steampowered.com/app/1174180/Red_Dead_Redemption_2/`. |
| `gameNames` | Array of strings | `[]` | Game names to search for; Steam's top search result for each name is scraped. |

At least one of `startUrls` or `gameNames` must be provided; you can mix both in the same run.

### Example: scrape by store URL

```json
{
  "startUrls": [
    { "url": "https://store.steampowered.com/app/1174180/Red_Dead_Redemption_2/" }
  ]
}
```

### Example: scrape by game name

```json
{
  "gameNames": ["Dota 2", "Hades II"]
}
```

### Example: mix of both

```json
{
  "startUrls": [
    { "url": "https://store.steampowered.com/app/570/Dota_2/" }
  ],
  "gameNames": ["Baldur's Gate 3"]
}
```

## Use cases

- **Price trackers** monitoring current price and discount percentage across a watchlist of games.
- **Game databases** backfilling genres, tags, requirements and media for a catalog.
- **Market researchers** comparing review scores and achievement counts across titles or genres.
- **Recommendation tools** pulling Steam's own "More like this" suggestions as a seed for further discovery.
- **Hardware compatibility tools** reading system requirements across platforms for a set of games.

## FAQ

**Does this need a Steam account?**
No. It reads the same public store pages and game details any visitor sees on steampowered.com.

**Does this work for free-to-play games?**
Yes. Free games return `isFree: true` and their price fields as `0`.

**Why does a field like `metacriticScore` or an award show as `0`, `""`, or an empty list?**
Every record always contains the same fields. When a value doesn't apply to that game (no Metacritic review, no awards), it's returned as an empty value rather than omitted.

**What happens if I list the same game by both URL and name?**
Games are deduplicated by Steam App ID, so it's only scraped and returned once.

**Why is `reviewScorePercent` a number from 1–9 instead of a percentage?**
That's Steam's own rating scale, where 9 means "Overwhelmingly Positive" and 1 means "Overwhelmingly Negative". The `reviewScore` field gives the human-readable label for the same value.

**How current is the pricing and review data?**
Every run fetches live data at request time, so prices and review counts reflect what Steam shows right now.

**Is this affiliated with Steam or Valve?**
No. This is an independent tool and is not affiliated with, endorsed by, or sponsored by Valve Corporation or Steam.
