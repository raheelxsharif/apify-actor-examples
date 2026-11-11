# Steam Game Specs, Pricing & Media Extractor

Steam shows each game through a store page plus an underlying app-details feed. This actor collects both for every title you give it and returns one clean record with the commercial, technical and promotional data a catalog needs: store pricing and package options, system requirements, platform and language support, screenshots and trailers, DLC, tags, regional age ratings, review totals and Metacritic information.

## Ways to select games

- **Store URLs** — paste full Steam store links, including age-gated links.
- **App IDs** — send numeric IDs or store links in any mix.
- **Name search** — search Steam by title and collect the top matches.
- **Steam charts** — walk top sellers, new releases, current specials, coming soon or weeklong deals.

All four paths feed the same detail collector, and duplicate App IDs are removed across the whole run. Add the optional app-type filter when you only want games, DLC, demos, software, video, music, series or episodes.

## What each record contains

| Group | Fields |
| --- | --- |
| Identity | `appId`, `appType`, `title`, `url`, `releaseDate`, `releaseDateIso`, `comingSoon`, `requiredAge`, `isFree`, `controllerSupport` |
| Pricing | `currency`, `priceInitial`, `priceFinal`, `priceInitialFormatted`, `priceFormatted`, `discountPercent`, `packages`, `packageGroups` |
| Descriptions | `shortDescription`, `detailedDescription`, `aboutTheGame`, `pressQuotes` |
| Artwork | `headerImage`, `capsuleImage`, `capsuleImageV5`, `backgroundImage`, `backgroundRawImage` |
| Media | `screenshots`, `screenshotThumbnails`, `trailers` (with HLS, DASH, MP4 and WebM links when Steam publishes them) |
| Requirements | `windowsRequirements`, `macRequirements`, `linuxRequirements` (`minimum` / `recommended`) |
| Support matrix | `platforms`, `supportedLanguages`, `fullyAudioLanguages` |
| Taxonomy | `genres`, `genreIds`, `categories`, `categoryIds`, `tags` |
| Credits and support | `developers`, `publishers`, `website`, `supportInfo` |
| Reviews | `totalReviews`, `positiveReviews`, `negativeReviews`, `reviewScore`, `reviewScorePercent`, `positiveReviewPercent`, `recommendationsTotal`, `metacriticScore`, `metacriticUrl` |
| Extras | `totalAchievements`, `dlcCount`, `dlcAppIds`, `dlc`, `contentDescriptors`, `contentDescriptorNotes`, `ratings`, `legalNotice`, `similarGames`, `awards`, `scrapedAt` |

Fields Steam genuinely does not publish for a particular app are left out of that record rather than filled with empty placeholders. That keeps every value real and makes schemas easier to validate.

`reviewScorePercent` is Steam's legacy 0–9 review-band index kept for backward compatibility; `positiveReviewPercent` is the actual share of positive reviews.

## Input reference

| Field | Type | Default | Purpose |
| --- | --- | --- | --- |
| `mode` | select | `urls` | `urls`, `appIds`, `search` or `browse`. |
| `startUrls` | URL list | – (prefill: Dota 2 page) | Steam store links for URL mode. |
| `appIds` | string list | – | Numeric App IDs or store links for ID mode. |
| `searchQueries` | string list | – | Game names for search mode. |
| `browseCategory` | select | `topSellers` | Chart used by browse mode. |
| `appType` | select | all | Keep only one Steam app type: game, DLC, demo, software, video, music, series, episode, hardware, mod or advertising. |
| `maxItems` | integer | `10` | Maximum game records to return (1–1000). |
| `maxResultsPerQuery` | integer | `10` | Search matches considered per name (1–100). |
| `country` | select | `us` | Steam storefront used for prices, availability and ratings. |
| `language` | select | `english` | Language for descriptions, requirements and review labels. |
| `includeReviews` | boolean | `true` | Fetch the current review summary. |
| `includeSimilarGames` | boolean | `true` | Include Steam's "More like this" games with titles. |

### URL mode

```json
{
  "mode": "urls",
  "startUrls": [
    { "url": "https://store.steampowered.com/app/1174180/Red_Dead_Redemption_2/" },
    { "url": "https://store.steampowered.com/app/1245620/ELDEN_RING/" }
  ],
  "country": "gb",
  "language": "english",
  "maxItems": 2
}
```

### App ID mode

```json
{
  "mode": "appIds",
  "appIds": ["570", "620", "https://store.steampowered.com/app/292030/"],
  "maxItems": 3
}
```

### Search mode

```json
{
  "mode": "search",
  "searchQueries": ["Hades II", "Baldur's Gate 3"],
  "maxResultsPerQuery": 3,
  "maxItems": 5,
  "appType": "game"
}
```

### Chart browsing

```json
{
  "mode": "browse",
  "browseCategory": "newReleases",
  "maxItems": 20,
  "country": "us"
}
```

## Backward compatibility

Existing tasks keep working. `startUrls` remains the native URL list; legacy `urls` and `appId` are mapped to their modern modes, and `gameNames` is mapped to name search. The old input keys are not shown in the form because the current form has clearer mode-based fields.

## Practical notes

- Prices are returned exactly as the chosen storefront reports them. Changing `country` changes both the currency and the available packages.
- `releaseDateIso` is built from Steam's release timestamp, so it stays consistent when you change `country` or `language`; the localized `releaseDate` text is returned alongside it. Year-only and quarter-only dates get no fabricated day.
- The review summary is a live snapshot. Counts move constantly on popular titles.
- Screenshots and trailers are direct Steam CDN links. Some trailer formats are streaming manifests (HLS/DASH) rather than single files; `videoUrl` picks the best published link and the format-specific fields are also included.
- DLC titles and links are read from the store page; `dlcAppIds` comes from the app-details feed. When both sources are present they are aligned by App ID.
- Similar-game titles cost one extra small lookup per recommendation; turn `includeSimilarGames` off for large chart runs if you only need the App IDs.
- No Steam account, cookie, API key or proxy configuration is needed.

## FAQ

**Can I mix URLs, IDs and names in one run?**
One mode runs per execution. Use URL or ID mode for exact titles, or search mode with several names; duplicates are removed automatically inside the chosen mode.

**Why is a price missing on a free game?**
Free titles carry `isFree: true` and zero price fields instead of a paid price block. Steam does not publish a currency price for them.

**Why do some games have no Metacritic fields?**
Steam only links Metacritic for games that have a matching review page. When the link is absent the fields are omitted.

**Does this actor download the videos or screenshots?**
No. It returns their CDN URLs so you can download only the media you need.

**Is this affiliated with Valve or Steam?**
No. This is an independent data tool and is not endorsed by or affiliated with Valve Corporation.
