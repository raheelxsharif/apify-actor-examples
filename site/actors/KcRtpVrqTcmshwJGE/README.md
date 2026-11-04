# Steam Game Reviews & Playtime Extractor

Pull the actual customer-review feed for any Steam game — the same text, thumbs up/down verdict, playtime, and helpfulness votes that shoppers read on a store page before buying. Point the actor at a store URL or just type the game's name, and it resolves the app, paginates through Steam's review feed, and returns one record per review with reviewer playtime and vote weight attached.

## What this actor does

- **Two ways to pick a game:** a Steam store page URL, or a plain-text game name that gets resolved to the matching store listing
- **Multi-game runs:** list several URLs or names in one job; each game is deduplicated by its Steam app ID and scraped independently
- **Review filters:** narrow to positive-only or negative-only reviews, and to a single Steam language code
- **Reviewer context on every row:** playtime at the time of review vs. total playtime, review count, and whether the copy was free or written during Early Access
- **Every field is always present** — Steam fields with no recorded value come back as `0`, `0.0`, `false`, or `""` rather than being left out

## Output per review

- `appId` — Steam application ID
- `gameTitle` — the matched store title when the game came from `gameNames`; when it came from `startUrls`, this is the Steam `appId` instead, since the actor doesn't fetch the store page title for direct URLs
- `gameUrl` — canonical store page URL
- `reviewId` — Steam's recommendation ID
- `reviewText` — full review body
- `rating` — `Positive` or `Negative` (from Steam's thumbs up/down)
- `authorSteamId` — reviewer's SteamID64
- `authorNumReviews` — total reviews this author has posted on Steam
- `authorPlaytimeForever` — total hours the author has played the game
- `authorPlaytimeAtReview` — hours played at the moment the review was written
- `votesHelpful` — "helpful" votes from other Steam users
- `votesFunny` — "funny" votes from other Steam users
- `weightedVoteScore` — Steam's internal helpfulness ranking (0–1)
- `timestampCreated` — when the review was posted (ISO 8601)
- `timestampUpdated` — when the review was last edited (ISO 8601)
- `receivedForFree` — whether the reviewer got the game for free
- `writtenDuringEarlyAccess` — whether the game was still in Early Access when reviewed
- `language` — Steam language code the review was written in
- `scrapedAt` — when this record was collected (ISO 8601)

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `startUrls` | array | – | Steam store page URLs to pull reviews for |
| `gameNames` | array | – | Game names to search; the top Steam search result is used |
| `maxReviews` | integer | `50` | Maximum reviews to collect per game (1–500) |
| `reviewType` | string | `all` | `all`, `positive`, or `negative` |
| `language` | string | `all` | Steam language code (`english`, `german`, `schinese`, etc.) or `all` for every language |

At least one of `startUrls` or `gameNames` must be provided.

### Example: reviews for a single game by URL

```json
{
    "startUrls": [{ "url": "https://store.steampowered.com/app/1174180/Red_Dead_Redemption_2/" }],
    "maxReviews": 100,
    "reviewType": "all",
    "language": "all"
}
```

### Example: negative English reviews only

```json
{
    "startUrls": [{ "url": "https://store.steampowered.com/app/252490/Rust/" }],
    "maxReviews": 200,
    "reviewType": "negative",
    "language": "english"
}
```

### Example: look up games by name instead of URL

```json
{
    "gameNames": ["Hades", "Baldur's Gate 3"],
    "maxReviews": 50,
    "reviewType": "positive"
}
```

## Use cases

- **Sentiment tracking** — monitor how the tone of reviews shifts after a patch, DLC launch, or price change
- **Competitor research** — compare review volume, rating mix, and complaint themes across similar games
- **QA and community management** — surface negative reviews mentioning bugs or crashes for triage
- **Influencer and playtester vetting** — check an author's total playtime and review history before trusting their opinion
- **Localization planning** — filter by language to see sentiment in specific markets before expanding support

## FAQ

**Where does this data come from?**
Steam's own public review feed for the app, the same data shown on a game's store page under "Customer Reviews."

**Do I need a Steam account or API key?**
No. The review feed is public and requires no login.

**What's the difference between using a URL and a game name?**
A URL points to an exact app ID, so it's always precise. A name runs a Steam search and uses the top match, which is usually correct but can occasionally resolve to the wrong edition (e.g. a bundle) for games with similar titles.

**Why is `authorPlaytimeAtReview` sometimes `0`?**
Steam doesn't record playtime-at-review for every review, particularly older ones or ones from accounts with private profiles; when that happens the field comes back as `0.0` rather than a real hours value.

**Can I scrape more than 500 reviews for one game?**
No, 500 per game per run is the current cap.

**What does `weightedVoteScore` mean?**
It's Steam's own ranking score that blends helpful and funny votes to estimate how useful a review is — higher values surface near the top of Steam's own review list.

**Can I combine URLs and game names in one run?**
Yes. Both lists are merged and deduplicated by app ID before scraping starts.

**Are language codes the same as ISO codes?**
No. Steam uses its own codes (e.g. `schinese`, `brazilian`, `latam`) rather than standard ISO 639-1 codes — use the values listed in the Input table.

**Is this affiliated with Steam or Valve?**
No. This is an independent, third-party actor that reads Steam's public review API; it isn't operated by Valve.
