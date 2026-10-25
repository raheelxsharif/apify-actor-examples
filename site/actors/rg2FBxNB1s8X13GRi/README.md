# Official MLB Team, Schedule & Stat Line Scraper

Pull baseball data straight from MLB's own public Stats API — season hitting, pitching, and fielding lines, team rosters of all 30 clubs, game-by-game schedules and results, and live division standings — in one of four focused modes.

## What this actor does

- Returns season-long player stat lines for hitting, pitching, or fielding, optionally narrowed to one team.
- Lists all 30 MLB teams with their league, division, home venue, and franchise details.
- Returns game-by-game schedule and results for any season or date range, including final scores.
- Returns live division standings with win/loss record, streak, run differential, and home/away splits.
- Fields the MLB Stats API doesn't return for a given record (for example a pitcher's `avg`-style hitting stats) are left out of that record.

## Output per mode

### Player stats (`getPlayerStats`)

- `playerId`, `playerName`, `teamId`, `teamName`, `position`, `season`
- `gamesPlayed`, `atBats`, `runs`, `hits`, `doubles`, `triples`, `homeRuns`, `rbi`, `baseOnBalls`, `stolenBases`
- `battingAverage`, `onBasePercentage`, `sluggingPct`, `ops` — hitting rate stats
- `wins`, `losses`, `saves`, `inningsPitched`, `earnedRuns`, `era`, `whip`, `strikeouts`, `strikeoutsPer9Inn`, `walksPer9Inn`, `winPercentage` — pitching stats
- `fieldingPercentage` — fielding stat
- `mlbUrl` — official MLB player page
- `scrapedAt`

### Teams (`getTeams`)

- `teamId`, `name`, `abbreviation`, `teamName`, `locationName`, `clubName`
- `firstYearOfPlay`, `teamCode`, `fileCode`
- `leagueName`, `leagueId`, `divisionName`, `divisionId`
- `venueName`, `venueId`
- `active`, `mlbUrl`, `scrapedAt`

### Schedule (`getSchedule`)

- `gamePk`, `gameDate`, `officialDate`, `gameType`, `status`
- `homeTeam`, `homeTeamId`, `awayTeam`, `awayTeamId`
- `homeScore`, `awayScore`, `homeWinner`
- `venue`, `mlbUrl`, `scrapedAt`

### Standings (`getStandings`)

- `teamName`, `teamId`, `leagueName`, `divisionName`, `season`
- `wins`, `losses`, `pct`, `divisionRank`, `leagueRank`, `wildCardRank`, `gamesBack`, `wildCardGamesBack`
- `streakCode`, `streakType`, `streakNumber`
- `runsScored`, `runsAllowed`, `runDifferential`
- `clinched`, `eliminationNumber`
- `homeRecord`, `awayRecord` — each `{wins, losses, pct}`
- `mlbUrl`, `scrapedAt`

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `getPlayerStats` | `getPlayerStats`, `getStandings`, `getSchedule`, or `getTeams` |
| `statGroup` | string | `hitting` | `hitting`, `pitching`, or `fielding` (mode=`getPlayerStats`) |
| `season` | integer | `2024` | MLB season year (2019–2024) |
| `teamId` | integer | – | Filter by MLB team ID (e.g. `147` = Yankees, `111` = Red Sox) |
| `startDate` | string | – | Start date `YYYY-MM-DD` (mode=`getSchedule`) |
| `endDate` | string | – | End date `YYYY-MM-DD` (mode=`getSchedule`) |
| `maxItems` | integer | `50` | Maximum number of records to return (1–500) |

### Example: hitting leaders for a season

```json
{
  "mode": "getPlayerStats",
  "statGroup": "hitting",
  "season": 2024,
  "maxItems": 30
}
```

### Example: one team's pitching stats

```json
{
  "mode": "getPlayerStats",
  "statGroup": "pitching",
  "season": 2024,
  "teamId": 147,
  "maxItems": 50
}
```

### Example: opening-week schedule

```json
{
  "mode": "getSchedule",
  "season": 2024,
  "startDate": "2024-03-20",
  "endDate": "2024-03-27",
  "maxItems": 50
}
```

### Example: current division standings

```json
{
  "mode": "getStandings",
  "season": 2024,
  "maxItems": 30
}
```

## Use cases

- **Fantasy baseball** — pull up-to-date batting and pitching lines to set a lineup
- **Betting models** — feed team standings and schedules into a predictive model
- **Sports apps** — power a scoreboard or standings widget with official MLB data
- **Analytics and research** — build season-over-season comparisons by team or player
- **Content and media** — generate recaps or leaderboards from structured stat lines

## FAQ

**Is this an official MLB product?**
No. This is an independent, third-party actor that reads MLB's public Stats API; it is not operated by or affiliated with MLB or any team.

**Do I need an API key?**
No — the MLB Stats API endpoints this actor uses are public and don't require authentication.

**Which seasons are supported?**
`season` accepts years 2019 through 2024.

**Why does `getPlayerStats` only show hitting stats for some players?**
`statGroup` selects one stat category per run; a player with no stats in that category for the season (e.g. a position player under `pitching`) simply won't appear.

**Can I get a single team's full schedule?**
Yes — set `teamId` alongside `mode: "getSchedule"` to scope the schedule to one club.

**Why is a field like `wildCardRank` missing from some standings records?**
The MLB Stats API only reports wild-card ranking for leagues/seasons where it applies; when it's absent from the source, the field is left out.

**What does `homeRecord`/`awayRecord` represent?**
A team's win-loss-percentage split for games played at home versus on the road, taken from the standings endpoint's own split records.

**How current is the data?**
As current as MLB's own Stats API — scores and standings update continuously during the season.
