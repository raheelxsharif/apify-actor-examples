# Multi-League Season Results & Team Lookup

Pull a full season of match results from OddsPortal — final scores, half-time scores, venues and 1X2 bookmaker odds — or switch to a league's team roster, or look a team up by name. Works for 30+ competitions across football, basketball, ice hockey, baseball, American football and tennis, plus any other OddsPortal league you paste in as a URL. No login, no API key, no browser.

## What this actor does

- **Season results & odds** — every match of a season with final score, winner, half-time score, kick-off (UTC), venue, status, plus 1X2 average and maximum odds, the number of bookmakers pricing the match and each bookmaker's price with its payout percentage.
- **League team roster** — the league table for a season: rank, matches played, wins, draws, losses, goals for/against, goal difference, points and recent form, one row per team.
- **Find a team by name** — search OddsPortal's team database (e.g. "Arsenal") and get every matching team with its country and page URL; optionally enrich each result with the team's official name, stadium, city, country and coach.
- **Any league, any season** — pick one of the 30 built-in competitions or paste any OddsPortal league or season-results URL. Historic archives go back 20+ seasons for major leagues (Premier League results reach 1998/99).
- **Multi-league runs** — pass several league URLs in one run and cap the output with Max items.

## Modes

| Mode | What you get |
|---|---|
| `results` | One row per match of the chosen league and season, with scores and odds. |
| `teams` | One row per team in the league table, with standings stats. |
| `teamSearch` | One row per team matching a name, with profile fields when enrichment is on. |

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | Dropdown | `results` | Season results & odds, league team roster, or find a team by name. |
| `sport` | Dropdown | `epl` | The competition to fetch (32 options — Premier League, NBA, NHL, MLB, NFL, Champions League, EuroLeague, Grand Slams and more). Ignored when League URLs is set. |
| `season` | Dropdown | `2024-2025` | Season to fetch. `2025-2026` and `2024-2025` style for soccer-style leagues, single years for calendar leagues (MLB, MLS, NPB, J1, K League), `current` for the live season. A plain year like `2024` expands automatically. |
| `leagueUrls` | String list | — | One or more OddsPortal league or season-results URLs, any sport. Takes priority over League and Season. A plain league URL gets `/results/` appended automatically. |
| `teamQuery` | String | — | Team name to look up (teamSearch mode), e.g. `Arsenal`. |
| `includeOdds` | Checkbox | true | Add 1X2 average/max odds, bookmaker count and per-bookmaker prices to each match. |
| `includeUpcoming` | Checkbox | true | Keep scheduled and in-progress matches; turn off for finished results only. |
| `enrichTeams` | Checkbox | true | Fetch each team's page for its official name, stadium, city, country, coach and badge (one extra request per team). |
| `regionalFallback` | Checkbox | true | When a league-season is empty on the main domain, retry it on the same site's regional edition (a few competitions, e.g. ACB, only publish older seasons there). Only fires on empty results. |
| `maxItems` | Integer | `20` | Maximum records to return (1–10,000). The run also stops when the season runs out of pages. |

### Example: a full Premier League season with odds

```json
{
    "mode": "results",
    "sport": "epl",
    "season": "2024-2025",
    "maxItems": 500,
    "includeOdds": true
}
```

### Example: NBA team roster with profiles

```json
{
    "mode": "teams",
    "sport": "nba",
    "season": "current",
    "maxItems": 30,
    "enrichTeams": true
}
```

### Example: any league by URL

```json
{
    "mode": "results",
    "leagueUrls": [
        "https://www.oddsportal.com/football/germany/bundesliga-2023-2024/results/"
    ],
    "maxItems": 100
}
```

## Output per match (mode: `results`)

| Field | Description |
|---|---|
| `eventId` | OddsPortal event ID (numeric string, stable across runs). |
| `eventName` | "Home vs Away". |
| `homeTeam` / `awayTeam` | Team names as shown in the results table. |
| `homeScore` / `awayScore` | Final score, present once the match is played. |
| `result` / `partialResult` | Score string (`2:1`) and period scores (`0:1, 1:1`, quarter scores for basketball). |
| `winner` | `home`, `draw` or `away`, when the match is finished. |
| `season` | Season label (`2024-2025`). |
| `matchDate` / `matchDateTime` | Kick-off date and full UTC timestamp. |
| `venue` | Stadium or court name, when recorded. |
| `status` | Finished, Scheduled, 1st Half, Half-time, … |
| `league` / `country` | Competition and country. |
| `sport` | The league key or sport you requested. |
| `homeTeamId` / `awayTeamId` | OddsPortal team IDs (join against team records). |
| `homeTeamUrl` / `awayTeamUrl` | Team page links. |
| `eventUrl` | Link to the match on OddsPortal. |
| `oddsHome` / `oddsDraw` / `oddsAway` | Average 1X2 odds (with `includeOdds`). |
| `oddsHomeMax` / `oddsDrawMax` / `oddsAwayMax` | Maximum available odds. |
| `bookmakerCount` | Number of bookmakers pricing the match. |
| `oddsBookmakers` | Per-bookmaker prices and payout percentages (nested). |
| `recordType` | Always `sport_event`. |
| `scrapedAt` | ISO 8601 fetch timestamp. |

## Output per team (modes: `teams`, `teamSearch`)

| Field | Description |
|---|---|
| `teamId` | OddsPortal team ID. |
| `teamName` | Team name as shown in tables. |
| `fullName` | Official club name (with enrichment). |
| `city` / `stadium` / `country` | Home location (with enrichment). |
| `coach` | Current coach, when recorded (with enrichment). |
| `logoUrl` | Team badge image. |
| `teamUrl` | Team page link. |
| `rank`, `matchesPlayed`, `wins`, `draws`, `losses` | League table position and record (roster mode). |
| `goalsFor`, `goalsAgainst`, `goalDifference`, `points` | Goals and points (roster mode). |
| `form` | Recent form, newest first (`["W","W","D"]`). |
| `league` / `sport` | Competition and requested key. |
| `recordType` | Always `sport_team`. |
| `scrapedAt` | ISO 8601 fetch timestamp. |

Fields the data source does not have for a record are omitted — never blank.

## Which leagues are covered?

The League dropdown ships 32 popular competitions: English Premier League, NFL, NBA, MLB, NHL, MLS, Bundesliga, Serie A, La Liga, Ligue 1, Champions League, Europa League, Conference League, Eredivisie, Liga Portugal, Super Lig, EFL Championship, Liga MX, Brasileirao, Argentine Liga Profesional, J1 League, K League 1, Saudi Pro League, EuroLeague, ACB, NCAA, KHL, NPB and the four Grand Slam tennis events. That is only a small slice of OddsPortal's catalog — for anything else paste the league or season URL into League URLs.

## FAQ

**Do I need a proxy, login or API key?** No. The actor fetches public OddsPortal pages and picks its own network route automatically.

**Does this include betting odds?** Yes. Every match row carries 1X2 average and maximum odds, the bookmaker count and (optionally) each bookmaker's price with payout percentage. Turn `includeOdds` off to keep rows lean.

**How far back does history go?** Most major leagues have archives going back 20+ seasons (the Premier League to 1998/99, NBA to the 1990s). A few competitions only expose the current season — the run's status message tells you when a requested season is unavailable.

**Why did one league return no rows?** OddsPortal sometimes answers a league-season page with "no odds available from your selected bookmakers" for anonymous visitors, and a few league-seasons are empty upstream on the main domain (ACB 2024-2025 is one). The actor retries through a fresh route, optionally tries the same site's regional edition (`regionalFallback`), validates that the served season is the one you asked for, and reports the exact reason in the status message instead of failing silently — it never returns another season's matches.

**Why does the bookmaker count vary between runs?** OddsPortal shows the bookmakers available in the location of the request. Runs on US-based exits typically see fewer bookmakers than runs from other regions; `bookmakerCount` and the per-bookmaker list show exactly what a match had, and odds are the real prices from that set.

**Do old seasons of renamed leagues work?** Yes — sponsor renames are mapped automatically (for example La Liga, the Brazilian Serie A, the Argentine league), so both current and historical season URLs resolve.

**Do upcoming matches have scores?** No — scheduled and in-progress matches carry kick-off time, status and odds; scores appear once the match is played. Use `includeUpcoming` to filter them out.

**What season format should I use?** Soccer-style leagues use `2024-2025` (a plain `2024` expands to `2024-2025`); calendar-year leagues (MLB, MLS, NPB, J1 League, K League) use `2024`; `current` always means the live season.

**How many records can I get per run, and can I include several leagues?** Up to 10,000 rows via Max items; the run stops early when a season's last page is reached. Put several URLs in League URLs — each is walked separately, and Max items is the total row budget across them.

**Why is there no short code like "NEW" for teams?** OddsPortal does not expose short team codes; that field is omitted instead of being guessed.

## Use cases

- **Season recaps and standings** — pull every result of a season to build tables, summaries or content.
- **Odds and market research** — export average, maximum and per-bookmaker 1X2 prices for a whole season.
- **Team reference data** — collect stadium, city, country, coach and badge details for every team in a league.
- **Historical analysis** — run once per season to compare results and odds across decades.
- **Data enrichment** — join `eventId`, `teamId` and team URLs against datasets you maintain.

## Limitations

- Full per-market books (over/under, Asian handicap, both teams to score) and opening/closing line history are not included in this actor — result rows carry the 1X2 market.
- A few competitions do not expose historical season links; older seasons for those are reported as unavailable rather than approximated.
- Team profile fields depend on what OddsPortal records for that team; smaller clubs may be missing venue or coach details.
- OddsPortal rate-limits bursts; the actor spaces requests and backs off automatically, which can make very large runs slower.

## Data source

OddsPortal (oddsportal.com), public pages only. The actor reads the site's own structured data from the league, season, standings, team and search pages and follows the site's own pagination. It never needs an account, cookies or a browser.
