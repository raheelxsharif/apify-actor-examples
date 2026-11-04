# Steam Game Reviews & Playtime Extractor

Export the actual customer-review feed for any Steam game — the review text, thumbs up/down verdict, reviewer profile, playtime, votes, and hardware shown on a store page. Pick games by app ID, store URL, community review URL, or plain name, then walk the full review history or stop at a fixed count. Every row is real Steam data; empty values are omitted instead of being filled with `0`, `false`, or `""`.

## What this actor does

- **Two output modes:** `reviews` emits one row per review; `gameSummary` emits one row per game with metadata, price, and the overall review score.
- **Flexible game selection:** numeric app IDs, store URLs, community review URLs, old review URLs, and game-name search all work in one run. Names prefer an exact title match.
- **Complete review filters:** language, review type, purchase type, sort order, helpfulness window, exact UTC date window, playtime range, Steam Deck, off-topic reviews, and review-quality ranking.
- **Rich reviewer context:** display name, profile and avatar URLs, SteamID, review counts, total/at-review/recent playtime, last played, Deck playtime, votes, comment count, reactions, purchase/refund/Early Access/Deck flags, and optional hardware details.
- **Unlimited history:** `maxReviews: 0` follows the review cursor until Steam runs out, with no 500-row ceiling.
- **Empty fields are omitted** at every nesting level, so a missing value is never confused with a real zero.

## Output per review (`mode` = `reviews`)

- `appId`, `gameTitle`, `gameUrl` — the game's Steam identity
- `reviewId`, `reviewText`, `reviewUrl` — Steam recommendation ID, body, and permalink
- `rating` — `Positive` or `Negative`; `votedUp` boolean form
- `authorSteamId`, `authorName`, `authorProfileUrl`, `authorAvatarUrl`
- `authorNumReviews`, `authorNumGamesOwned` — profile activity (games owned is omitted when private or zero)
- `authorPlaytimeForever`, `authorPlaytimeAtReview`, `authorPlaytimeLastTwoWeeks`, `authorDeckPlaytimeAtReview` — hours, one decimal
- `authorLastPlayed` — ISO 8601 UTC
- `votesHelpful`, `votesFunny`, `weightedVoteScore`, `commentCount`, `reactionCount`
- `steamPurchase`, `receivedForFree`, `refunded`, `writtenDuringEarlyAccess`, `primarilySteamDeck`
- `language`, `timestampCreated`, `timestampUpdated` — Steam language code and ISO 8601 UTC timestamps
- `developerResponse`, `developerResponseAt` — only when Steam returns a developer reply
- `hardware` — optional `{ os, cpu, gpu, systemRamMb, vramMb }`
- `game` — optional metadata block when `includeGameDetails` is on
- `recordType: "review"`, `scrapedAt`

## Output per game (`mode` = `gameSummary`)

- `appId`, `gameTitle`, `gameUrl`, `gameType`, `isFree`, `shortDescription`
- `developers[]`, `publishers[]`, `genres[]`, `categories[]`
- `platforms` — `{ windows, mac, linux }`
- `releaseDate`, `metacriticScore`, `headerImage`
- `price` — `{ currency, initial, final, discountPercent, formatted }` when a price exists
- `reviewSummary` — `{ reviewScore, reviewScoreDesc, totalPositive, totalNegative, totalReviews, positivePercent }`
- `recordType: "game"`, `scrapedAt`

`reviewSummary` is the game's overall score across all languages and purchase types, not the filtered review query.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `reviews` | `reviews` (one row per review) or `gameSummary` (one row per game) |
| `appIds` | array | – | Numeric IDs, store URLs, community review URLs, or old review URLs |
| `startUrls` | array | – | Legacy-compatible Steam store URLs; merged with `appIds` |
| `gameNames` | array | – | Names resolved through Steam search; exact title match preferred |
| `maxReviews` | int | `50` | Per-game review cap; `0` means all matching reviews |
| `maxGames` | int | `0` | Cap the merged game list; `0` means no cap |
| `reviewType` | string | `all` | `all`, `positive`, or `negative` |
| `language` | string | `all` | Steam language code or `all` |
| `sortBy` | string | `recent` | `recent`, `helpful`, `updated`, or `funny`; helpful and funny are ranked, not exhaustive |
| `helpfulWithinDays` | int | `30` | Helpful sort window (1–365 days) |
| `purchaseType` | string | `all` | `all`, `steam`, or `nonSteamPurchase` |
| `minPlaytimeHours` | int | `0` | Minimum hours at review; `0` = no minimum |
| `maxPlaytimeHours` | int | `0` | Maximum hours at review; `0` = no maximum |
| `steamDeckOnly` | boolean | `false` | Only reviews played primarily on Steam Deck |
| `includeOffTopic` | boolean | `false` | Include review-bomb/off-topic activity Steam hides by default |
| `useReviewQuality` | boolean | `true` | Apply Steam's quality ranking to `sortBy=helpful` |
| `reviewsFromDate` | string | – | Include reviews created on/after this UTC date (`YYYY-MM-DD`) |
| `reviewsToDate` | string | – | Include reviews created on/before this UTC date (`YYYY-MM-DD`) |
| `includeGameDetails` | boolean | `false` | Attach game metadata, price, and score to every review row |

At least one game source (`appIds`, `startUrls`, or `gameNames`) is required.

### Example: the smallest daily run

```json
{
    "mode": "reviews",
    "startUrls": [{ "url": "https://store.steampowered.com/app/1174180/Red_Dead_Redemption_2/" }],
    "maxReviews": 3,
    "sortBy": "recent"
}
```

### Example: negative English reviews in an exact date window

```json
{
    "mode": "reviews",
    "appIds": ["570"],
    "reviewType": "negative",
    "language": "english",
    "reviewsFromDate": "2024-01-01",
    "reviewsToDate": "2024-03-31",
    "maxReviews": 500
}
```

### Example: long sessions on a Steam Deck

```json
{
    "mode": "reviews",
    "gameNames": ["Hades"],
    "steamDeckOnly": true,
    "minPlaytimeHours": 20,
    "sortBy": "helpful",
    "helpfulWithinDays": 90,
    "maxReviews": 200
}
```

### Example: game metadata and score summaries

```json
{
    "mode": "gameSummary",
    "appIds": ["1145360", "1245620"],
    "gameNames": ["Baldur's Gate 3"],
    "maxGames": 3
}
```

## Use cases

- **Patch and DLC sentiment** — pull reviews from a narrow date window to see reaction before and after a release.
- **Competitive benchmarking** — compare review volume, positive share, playtime, and hardware mix across titles.
- **Community management** — filter negative reviews by language and date to triage bugs and crashes.
- **Reviewer vetting** — check total playtime, review history, and whether a copy was free or refunded.
- **Store dashboards** — run `gameSummary` for a clean one-row-per-game view of price, score, and metadata.
- **Localization research** — compare sentiment in specific Steam language markets.

## FAQ

**Where does the data come from?**
Steam's public store review services and store metadata endpoints — the same information shown under "Customer Reviews" and on a game's store page.

**Do I need a Steam account, API key, or proxy?**
No. The actor uses anonymous public endpoints, needs no login or key, and is configured without a proxy input.

**Are the date filters inclusive?**
Yes. `reviewsFromDate` starts at 00:00:00 UTC and `reviewsToDate` ends at 23:59:59 UTC, using the review's creation time. Reviews edited later do not move into an older window.

**Why are some fields missing from a row?**
Empty values are omitted deliberately. Hardware is only recorded when a reviewer shares it, reactions and developer responses are rare, and private profiles hide games-owned. If Steam's rich review service is unavailable, the actor switches to Valve's official review API, which omits reviewer display names and avatars. Absence means Steam did not provide the value.

**Does `maxReviews: 0` really return everything?**
With `recent` or `updated` it follows Steam's cursor until the source is exhausted. Ranked sorts (`helpful`, `funny`) only expose Steam's ranked subset. A safety cap of 1,000,000 reviews per game prevents runaway runs; narrow the date window or use `maxReviews` for very large titles.

**Is `sortBy=helpful` a full export?**
No. Helpful is Steam's ranked view inside `helpfulWithinDays`, and `funny` is likewise a ranked subset of the game's reviews; neither is exhaustive. Use `recent` or `updated` for full-history exports.

**What does `purchaseType` mean?**
`steam` means Steam records the copy as bought through Steam; `nonSteamPurchase` covers keys, gifts, and other acquisitions. On some free-to-play titles Steam's own purchase flag does not split cleanly, so treat that filter as Steam's classification rather than an independent audit.

**Why does `gameSummary` show a different score than a filtered review run?**
`reviewSummary` is always the game's overall score across all languages and purchase types. The review filters only affect which individual review rows are emitted.

**Can I mix IDs, URLs, and names in one run?**
Yes. All sources are merged and deduplicated by app ID, then capped by `maxGames`.

**Is this affiliated with Steam or Valve?**
No. This is an independent, third-party actor that reads Steam's public endpoints; it is not operated by Valve.
