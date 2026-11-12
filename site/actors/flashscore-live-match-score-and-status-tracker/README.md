# FlashScore Live Scores, Fixtures, Results & Standings Scraper

Collect FlashScore match data in four ways: a live/daily board for one sport, upcoming fixtures for a competition, recent results, and full league tables. Every record is plain JSON with teams, scores, status, kickoff time, league context and links back to FlashScore. No login, cookies, API key or proxy setup is required.

## Modes

| Mode | What it returns | Main input |
|---|---|---|
| `live` | Every match on a day for one sport, optionally filtered to in-play games | `sport`, `dayOffset`, `liveOnly` |
| `fixtures` | Upcoming matches for one or more competitions, ordered by kickoff | `leagueUrls` |
| `results` | Recent finished matches for one or more competitions, newest first | `leagueUrls` |
| `standings` | Overall, home or away league table for one or more competitions | `leagueUrls`, `standingsType` |

The default run is the live/daily board for football, which returns the smallest useful dataset and needs a single request.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `live` | `live`, `fixtures`, `results` or `standings` |
| `sport` | string | `football` | One of 14 sports for live mode |
| `dayOffset` | integer | `0` | Days from today: `-7` to `7` |
| `liveOnly` | boolean | `true` | In live mode, keep only matches in progress |
| `leagueUrls` | array | `[]` | Competition URLs for fixtures, results and standings (up to 25) |
| `standingsType` | string | `overall` | `overall`, `home` or `away` |
| `maxItems` | integer | `100` | Hard cap across the whole run (1–1000) |

Providing competition URLs selects `fixtures` when `mode` is left at its default. Set `mode` to `results` or `standings` when you want those competition feeds instead.

### Legacy inputs

Saved tasks from the earlier live-only actor keep working without schema changes. Inputs without `mode` still run the daily live board. These aliases are read in code:

| Current field | Legacy aliases |
|---|---|
| `leagueUrls` | `leagueUrl`, `leagues`, `urls` |
| `standingsType` | `tableType` |
| `dayOffset` | `day`, `day_offset` |
| `liveOnly` | `live_only` |
| `maxItems` | `max_items` |

`proxyConfiguration`, `proxy` and `useProxy` are accepted but ignored; the actor is cookieless and always connects directly.

### Example: live football now

```json
{
    "mode": "live",
    "sport": "football",
    "liveOnly": true,
    "maxItems": 50
}
```

### Example: tomorrow's tennis schedule

```json
{
    "mode": "live",
    "sport": "tennis",
    "dayOffset": 1,
    "liveOnly": false,
    "maxItems": 100
}
```

### Example: Premier League fixtures and results

```json
{
    "mode": "fixtures",
    "leagueUrls": [
        "https://www.flashscore.com/football/england/premier-league/"
    ],
    "maxItems": 100
}
```

### Example: NBA standings by conference

```json
{
    "mode": "standings",
    "leagueUrls": [
        "https://www.flashscore.com/basketball/usa/nba/"
    ],
    "standingsType": "overall",
    "maxItems": 100
}
```

## Output

Match records carry `recordType: "match"` and these fields when FlashScore provides them:

- `matchId`, `mode`, `sport`, `status`, `statusCode`, `startTime`
- `homeTeam`, `awayTeam`, `homeScore`, `awayScore`
- `homeTeamId`, `awayTeamId`, `homeTeamSlug`, `awayTeamSlug`
- `homeTeamShort`, `awayTeamShort`, `homeTeamLogo`, `awayTeamLogo`
- `homeTeamUrl`, `awayTeamUrl`, `url`
- `league`, `country`, `leaguePath`, `leagueUrl`, `round`
- `periodCode` and `period` for in-play football (first/second half)
- `scrapedAt`

Standing records carry `recordType: "standing"` and these fields when available:

- `league`, `country`, `leaguePath`, `leagueUrl`, `tableType`
- `group`, `tableName`, `rank`, `team`, `teamId`, `teamUrl`, `teamLogo`
- `played`, `wins`, `draws`, `losses`
- `scoredFor`, `scoredAgainst`, `scoreDifference`
- `points`, `winPercentage`, `form`
- `scrapedAt`

`form` lists the most recent results first. `scoredFor` and `scoredAgainst` are goals in football and hockey, points in basketball, and runs in cricket. Empty fields are omitted; a scheduled match has no score fields and a competition with no table has no standing rows.

## Pagination and coverage

- Live mode reads one day-board feed per sport. `dayOffset` moves from yesterday to a week ahead.
- Fixtures mode follows FlashScore's own page parameter and de-duplicates matches by `matchId`, so it can return the whole remaining schedule of a competition.
- Results mode returns the latest 100 finished matches FlashScore exposes for the competition season. For older seasons, add that season's archive URL, for example `https://www.flashscore.com/football/england/premier-league-2025-2026/`.
- Standings mode reads the complete table, including conference/group sections when FlashScore splits it.
- Multiple competition URLs are processed in the order given; `maxItems` is shared across all of them.

## Common questions

**Does this need a FlashScore account or cookies?**
No. The actor reads the same public match feeds and league pages that FlashScore serves to anonymous visitors.

**Can it run on a schedule?**
Yes. Live mode is a single request per run, so a short schedule is cheap. Each run reflects the score and status at the moment FlashScore publishes them.

**Why did a competition URL return nothing?**
Some competitions have no upcoming fixtures, no results in the current season, or no table at all (for example a finished knockout stage). Check the URL in a browser first; the actor also reports skipped URLs in the run status.

**Why is a score missing?**
A match that has not started has no score, so the field is omitted rather than reported as zero.

**Can I get more than 100 results for a season?**
FlashScore's results feed exposes the latest 100 matches per competition season. Use a past-season archive URL for older matches, or run the actor daily with mode=results to build a rolling archive.

**Are match details like lineups, cards or statistics included?**
No. This actor focuses on match-level scores, fixtures, results and standings. It does not open every match page for play-by-play data.

**Is this affiliated with FlashScore?**
No. It is an independent third-party tool that reads FlashScore's public data. FlashScore is a trademark of its owner.

## Data source

FlashScore's public daily feeds, tournament fixture/results feeds, standings feeds and league pages. The actor works without a browser or proxy and uses TLS impersonation only to present a normal Chrome request profile.

## Notes for automation

- Use `recordType` to split a mixed dataset: `match` or `standing`.
- `matchId` is stable across runs and is the natural key for de-duplication.
- `statusCode` is FlashScore's raw status: `1` scheduled, `2` live, `3` finished. `status` adds a readable label.
- `startTime` is ISO 8601 in UTC.
