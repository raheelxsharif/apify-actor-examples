# Official MLB Team, Schedule & Stat Line Scraper

Pull Major League Baseball data straight from MLB's public Stats API. Get all 30 clubs, player bios and rosters, schedules with probable pitchers and final scores, division standings, season and career stat lines, player game logs, per-player box scores, league leaderboards, and ballpark reference data. Nine modes, one dataset, no API key, no login, and no proxy setup.

## What this actor does

- **Nine modes in one actor** — teams, players/rosters, player stats, game logs, schedule, standings, box scores, leaders, and reference data.
- **Complete player pool** — season stats include every player the source returns, not only qualified hitters or pitchers.
- **Historical coverage** — stats reach back to 1876 and schedules to 1901; request one season or several in the same run.
- **Game-level detail** — schedules include probable pitchers, weather, decisions, attendance, umpires, and inning line scores; box scores return per-player batting, pitching, or fielding lines.
- **Clean, flat records** — every record carries a `recordType` and `scrapedAt`; fields the source does not have for a record are omitted rather than sent as null.

## Modes

| Mode | One record per | Highlights |
|---|---|---|
| `getTeams` | Club | League, division, venue, franchise, logo, team URL |
| `getPlayers` | Player | Bio, position, bats/throws, debut, headshot, team; rosters and name search; optional season stats |
| `getPlayerStats` | Player stat split | Season, career, season-by-season, month, day-of-week, home/away, or date-range lines |
| `getGameLog` | Player per game | One row per game with the full stat line, opponent, and game ID |
| `getSchedule` | Game | Scores, status, probable pitchers, weather, decisions, attendance, umpires, line score |
| `getStandings` | Team season record | Wins, losses, pct, ranks, games back, streaks, splits, clinch and elimination numbers |
| `getBoxscore` | Player per game | Batting, pitching, or fielding line plus game context; optional team totals and line score |
| `getLeaders` | Ranked player | Category, rank, value, player, team, league |
| `getReference` | Venue / league / division / season | Ballpark capacity, surface, roof, coordinates, league and division metadata, season calendar |

## Output per record type

### Team (`getTeams`)

`recordType`, `teamId`, `name`, `abbreviation`, `teamName`, `shortName`, `locationName`, `clubName`, `franchiseName`, `firstYearOfPlay`, `teamCode`, `fileCode`, `leagueName`, `leagueId`, `divisionName`, `divisionId`, `venueName`, `venueId`, `sportName`, `springLeagueName`, `allStarStatus`, `active`, `season`, `logoUrl`, `mlbUrl`, `scrapedAt`.

### Player (`getPlayers`)

`recordType`, `playerId`, `fullName`, `firstName`, `lastName`, `middleName`, `useName`, `primaryNumber`, `birthDate`, `currentAge`, `birthCity`, `birthStateProvince`, `birthCountry`, `height`, `weight`, `active`, `primaryPosition`, `primaryPositionCode`, `primaryPositionType`, `batSide`, `pitchHand`, `mlbDebutDate`, `draftYear`, `nickName`, `nameSlug`, `boxscoreName`, `currentTeam`, `currentTeamId`, `leagueName`, `divisionName`, `headshotUrl`, `mlbUrl`, `season`. Roster rows also include `jerseyNumber`, `rosterStatus`, `parentTeamId`, `rosterPosition`, and `rosterPositionAbbreviation`. With `includeStats`, a `stats` object holds the selected stat group.

### Player stats (`getPlayerStats`)

`recordType`, `playerId`, `playerName`, `fullName`, `firstName`, `lastName`, `teamId`, `teamName`, `position`, `positionAbbreviation`, `leagueName`, `sportId`, `season`, `statType`, `statGroup`, `splitKey`, `splitValue`, `month`, `monthName`, `dayOfWeek`, `dayOfWeekName`, `isHome`, `situationCode`, `startDate`, `endDate`, `mlbUrl`. Every non-empty stat field returned by MLB is included, for example: `gamesPlayed`, `atBats`, `runs`, `hits`, `doubles`, `triples`, `homeRuns`, `rbi`, `baseOnBalls`, `walks`, `strikeOuts`, `strikeouts`, `stolenBases`, `caughtStealing`, `avg`, `battingAverage`, `obp`, `onBasePercentage`, `slg`, `sluggingPct`, `ops`, `babip`, `wins`, `losses`, `saves`, `holds`, `blownSaves`, `inningsPitched`, `earnedRuns`, `era`, `whip`, `battersFaced`, `numberOfPitches`, `completeGames`, `shutouts`, `strikeoutsPer9Inn`, `walksPer9Inn`, `fieldingPercentage`, `chances`, `putOuts`, `assists`, `errors`, `doublePlays`.

### Game log (`getGameLog`)

`recordType`, `playerId`, `playerName`, `fullName`, `teamId`, `teamName`, `opponentId`, `opponentName`, `isHome`, `gamePk`, `officialDate`, `date`, `season`, `gameType`, `statGroup`, `mlbUrl`, plus every non-empty stat field for that game.

### Game (`getSchedule`)

`recordType`, `gamePk`, `gameGuid`, `season`, `gameDate`, `officialDate`, `gameType`, `gameNumber`, `doubleHeader`, `dayNight`, `seriesDescription`, `seriesGameNumber`, `gamesInSeries`, `scheduledInnings`, `isTie`, `recordSource`, `publicFacing`, `status`, `abstractGameState`, `codedGameState`, `statusCode`, `homeTeam`, `homeTeamId`, `homeTeamAbbreviation`, `awayTeam`, `awayTeamId`, `awayTeamAbbreviation`, `homeScore`, `awayScore`, `homeWinner`, `awayWinner`, `venue`, `venueId`, `awayProbablePitcher`, `awayProbablePitcherId`, `homeProbablePitcher`, `homeProbablePitcherId`, `weatherCondition`, `weatherTemp`, `weatherWind`, `awayRuns`, `homeRuns`, `awayHits`, `homeHits`, `awayErrors`, `homeErrors`, `innings`, `winningPitcher`, `winningPitcherId`, `losingPitcher`, `losingPitcherId`, `savePitcher`, `savePitcherId`, `attendance`, `firstPitch`, `gameDurationMinutes`, `delayDurationMinutes`, `officials`, `noHitter`, `perfectGame`, `mlbUrl`, `scrapedAt`.

### Standing (`getStandings`)

`recordType`, `teamName`, `teamId`, `leagueName`, `leagueId`, `divisionName`, `divisionId`, `season`, `wins`, `losses`, `gamesPlayed`, `pct`, `divisionRank`, `leagueRank`, `sportRank`, `wildCardRank`, `gamesBack`, `wildCardGamesBack`, `leagueGamesBack`, `sportGamesBack`, `divisionGamesBack`, `streakCode`, `streakType`, `streakNumber`, `runsScored`, `runsAllowed`, `runDifferential`, `clinched`, `divisionChamp`, `divisionLeader`, `hasWildcard`, `clinchIndicator`, `magicNumber`, `eliminationNumber`, `wildCardEliminationNumber`, `homeRecord`, `awayRecord`, `lastTenRecord`, `overallRecord`, `leagueRecord`, `divisionRecord`, `mlbUrl`, `scrapedAt`.

### Box score (`getBoxscore`)

`recordType` (`battingLine`, `pitchingLine`, `fieldingLine`), `statGroup`, `gamePk`, `season`, `gameDate`, `officialDate`, `gameType`, `status`, `homeTeam`, `homeTeamId`, `awayTeam`, `awayTeamId`, `homeScore`, `awayScore`, `venue`, `attendance`, `weatherCondition`, `weatherWind`, `firstPitch`, `gameDurationMinutes`, `winningPitcher`, `losingPitcher`, `savePitcher`, `isHome`, `teamName`, `teamId`, `playerId`, `playerName`, `fullName`, `jerseyNumber`, `position`, `battingOrder`, `summary`, `decision`, `mlbUrl`, and every non-empty stat field for the selected group. When games come from `gamePks`, only fields the boxscore payload carries are present (for example `officialDate`, not `gameDate`, `gameType`, or `status`). `seasonBattingAverage`, `seasonOnBasePercentage`, `seasonSluggingPct`, `seasonOps`, `seasonEra`, and `seasonWhip` add season context. With `includeTeamTotals`, `teamGame` records add `teamStats` and `lineScore`.

### Leader (`getLeaders`)

`recordType`, `leaderCategory`, `statGroup`, `season`, `gameType`, `gameTypeDescription`, `rank`, `value`, `playerId`, `playerName`, `teamId`, `teamName`, `leagueName`, `sportId`, `mlbUrl`, `scrapedAt`.

### Reference (`getReference`)

- Venue: `recordType`, `venueId`, `name`, `active`, `season`, `city`, `state`, `stateAbbrev`, `postalCode`, `country`, `address`, `phone`, `latitude`, `longitude`, `azimuthAngle`, `elevation`, `capacity`, `turfType`, `roofType`, outfield distances (`leftLine`, `left`, `leftCenter`, `center`, `rightCenter`, `right`, `rightLine`).
- League: `recordType`, `leagueId`, `name`, `abbreviation`, `nameShort`, `seasonState`, `hasWildCard`, `hasSplitSeason`, `numGames`, `numTeams`, `numWildcardTeams`, and season calendar dates.
- Division: `recordType`, `divisionId`, `name`, `nameShort`, `abbreviation`, `leagueId`, `leagueName`, `sportId`, `hasWildcard`, `active`, `sortOrder`, `numPlayoffTeams`.
- Season: `recordType`, `seasonId`, `hasWildcard`, and the full MLB season calendar (`regularSeasonStartDate`, `regularSeasonEndDate`, `postSeasonStartDate`, `postSeasonEndDate`, and related dates).

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `getPlayerStats` | `getTeams`, `getPlayers`, `getPlayerStats`, `getGameLog`, `getSchedule`, `getStandings`, `getBoxscore`, `getLeaders`, `getReference` |
| `seasons` | array | – | Season years, e.g. `2026`; multiple seasons allowed |
| `teamIds` | array | all teams | MLB team IDs, names, abbreviations, or team URLs (e.g. `147`, `Yankees`, `NYY`) |
| `playerIds` | array | – | MLB player IDs, e.g. `660271` |
| `gamePks` | array | – | MLB game IDs for `getBoxscore` |
| `startDate` | string | – | `YYYY-MM-DD` for schedules, box scores, or date-range stats |
| `endDate` | string | – | `YYYY-MM-DD` for schedules, box scores, or date-range stats |
| `statGroup` | string | `hitting` | `hitting`, `pitching`, or `fielding` |
| `statType` | string | `season` | `season`, `career`, `yearByYear`, `byMonth`, `byDayOfWeek`, `homeAndAway`, `byDateRange` |
| `rosterType` | string | `active` | `active`, `40Man`, `fullSeason`, `depthChart`, `coach` |
| `searchQuery` | string | – | Player name search for `getPlayers`, e.g. `Ohtani` |
| `leagueId` | string | `both` | `both`, `AL`, or `NL` for standings and leaders |
| `gameType` | string | `R` | `S`, `R`, `F`, `D`, `L`, `W`, `C`, `P`, `A`, `I`, `E` |
| `leaderCategory` | string | `homeRuns` | Category for `getLeaders`, e.g. `homeRuns`, `battingAverage`, `earnedRunAverage`, `saves` |
| `referenceType` | string | `venues` | `venues`, `leagues`, `divisions`, or `seasons` |
| `includeStats` | boolean | `false` | Add season stats to `getPlayers` rows |
| `includeTeamTotals` | boolean | `false` | Add team totals and line score to `getBoxscore` |
| `maxItems` | integer | `50` | Hard cap on records (1–10,000) |

## Examples

### A team's roster with season stats

```json
{
  "mode": "getPlayers",
  "seasons": ["2026"],
  "teamIds": ["Yankees"],
  "rosterType": "active",
  "includeStats": true,
  "maxItems": 40
}
```

### A week of games with probable pitchers and decisions

```json
{
  "mode": "getSchedule",
  "startDate": "2026-06-01",
  "endDate": "2026-06-07",
  "gameType": "R",
  "maxItems": 100
}
```

### Season pitching stats for two players

```json
{
  "mode": "getPlayerStats",
  "seasons": ["2024", "2025"],
  "playerIds": ["660271", "592450"],
  "statGroup": "pitching",
  "statType": "season",
  "maxItems": 20
}
```

### A box score with per-player lines and team totals

```json
{
  "mode": "getBoxscore",
  "gamePks": ["822974"],
  "statGroup": "hitting",
  "includeTeamTotals": true,
  "maxItems": 50
}
```

## Use cases

- **Fantasy baseball** — pull rosters, season lines, and per-game logs for lineup decisions.
- **Betting and prediction models** — backtest with schedules, scores, probable pitchers, and box scores.
- **Sports apps and dashboards** — power scoreboards, standings tables, and player pages.
- **Analytics and research** — join stable MLB IDs across seasons, teams, and games.
- **Media and newsletters** — generate recaps, leaderboards, and matchup previews.
- **Data engineering** — build clean historical datasets with one record type per row.

## FAQ

### Is this an official MLB product?

No. This is an independent, third-party actor that reads MLB's public Stats API. It is not affiliated with, endorsed by, or sponsored by Major League Baseball or any club.

### Do I need an API key, login, or proxy?

No. The public Stats API needs no key or account, and the actor does not require a proxy.

### How far back does the data go?

Player stats go back to 1876 and schedules to 1901 in the source. Early seasons have fewer recorded fields; those fields are omitted when the source does not have them.

### What is the difference between `getPlayerStats`, `getGameLog`, and `getBoxscore`?

`getPlayerStats` returns season, career, or split totals. `getGameLog` returns one row per player per game. `getBoxscore` returns every player's batting, pitching, or fielding line from one game, plus game context.

### Why are some fields missing from a record?

The API does not publish every field for every player, game, or season. Empty fields are omitted rather than filled with placeholders.

### Why is there no save pitcher on some games?

A save only exists when the game situation creates one. Games without a save omit `savePitcher`.

### Can I request several seasons or teams at once?

Yes. Put multiple years in `seasons` and multiple IDs, names, or team URLs in `teamIds`. Duplicate rows are removed by MLB IDs.

### Which seasons need player or team IDs for splits?

Season, career, and date-range stats work for the full player pool. Date-range stats take their seasons from `startDate`/`endDate`, so the dates and `seasons` do not disagree. Season-by-season, month, day-of-week, and home/away splits require `playerIds` or `teamIds` because they are per-player views in the source.

### Will my existing tasks keep working?

Yes. Existing modes and output fields are preserved. Older tasks that send `season` or `teamId` are still accepted and mapped to the new fields.

### Are doubleheaders handled?

Yes. Games are de-duplicated by `gamePk`, so both games of a doubleheader are kept.

## Data source

The actor reads `statsapi.mlb.com`, the public JSON feed behind MLB.com and the MLB app. It is an undocumented public API; endpoint behavior and field availability can change. Use the data in accordance with MLB's terms and any applicable law.
