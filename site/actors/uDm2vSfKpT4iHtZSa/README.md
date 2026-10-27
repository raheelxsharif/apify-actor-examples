# Transfermarkt Player Market Value & Transfer Scraper

Transfermarkt tracks market values, transfer fees, and squad data for football players and clubs worldwide. This actor searches that data by name or pulls a full record by ID — a player's transfer and injury history, a club's current squad, or every club in a competition.

## What this actor does

- Searches for players, clubs, or competitions by name.
- Pulls a single player's full profile by ID, including market value history, transfer records, and injury log.
- Pulls a club's profile plus its full current squad by ID, or every club competing in a given competition.
- Filters player results by minimum market value, nationality, or position.
- Fields Transfermarkt doesn't publish for a given record (an unlisted market value, a missing stadium name) are left out rather than sent empty.

## Output per record type

### Player

- `playerId` — Transfermarkt numeric player ID
- `name`, `fullName` — display and full name
- `nationality` — array of nationalities
- `position`, `subPosition` — main and secondary playing position
- `dateOfBirth`, `age`
- `height`, `foot`
- `currentClub`, `currentClubId`
- `marketValue` — formatted current market value (e.g. `€15.00m`)
- `marketValueNumeric` — the same value in euros as a number
- `highestMarketValue`, `highestMarketValueDate` — peak market value and when it was recorded (full profile only, mode `byPlayerId`)
- `transfers` — array of `{date, fromClub, toClub, fee, feeNumeric}` (full profile only)
- `injuries` — array of `{date, injury, absence}` (full profile only)
- `profileUrl`, `imageUrl`
- `siteName` — always `Transfermarkt`
- `recordType` — always `player`
- `scrapedAt` — UTC timestamp of the fetch

### Club

- `clubId` — Transfermarkt numeric club ID
- `name`, `officialName`
- `country`
- `league`, `leagueId`
- `squadSize`, `avgAge`, `foreignersNumber`
- `totalMarketValue`
- `stadiumName`, `stadiumSeats`
- `profileUrl`, `imageUrl`
- `siteName` — always `Transfermarkt`
- `recordType` — always `club`
- `scrapedAt` — UTC timestamp of the fetch

### Competition

- `competitionId` — Transfermarkt competition ID
- `name`, `country`, `countryId`
- `competitionUrl`, `imageUrl`
- `siteName` — always `Transfermarkt`
- `recordType` — always `competition`
- `scrapedAt` — UTC timestamp of the fetch

Running `byClubId` returns the club record followed by one player record per squad member; `byCompetitionId` returns one club record per club in that competition.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `searchPlayers` | `searchPlayers`, `byPlayerId`, `searchClubs`, `byClubId`, `searchCompetitions`, or `byCompetitionId` |
| `query` | string | `Messi` | Name to search for; used by the search modes |
| `playerId` | string | – | Transfermarkt numeric player ID, e.g. `28003` for Messi; used by `byPlayerId` |
| `clubId` | string | – | Transfermarkt numeric club ID, e.g. `131` for Manchester City; used by `byClubId` |
| `competitionId` | string | – | Transfermarkt competition ID, e.g. `GB1` for Premier League; used by `byCompetitionId` |
| `minMarketValueEur` | integer | – | Only keep players with a market value at or above this amount in euros |
| `nationality` | string | – | Only keep players whose nationality contains this text, e.g. `Argentina` |
| `position` | string | (any) | Only keep players in this exact position |
| `maxItems` | integer | `5` | Maximum number of records to return (1-1000) |

### Example: search for a player

```json
{
  "mode": "searchPlayers",
  "query": "Messi",
  "maxItems": 10
}
```

### Example: full player profile with transfer and injury history

```json
{
  "mode": "byPlayerId",
  "playerId": "28003"
}
```

### Example: club profile and squad, filtered to forwards

```json
{
  "mode": "byClubId",
  "clubId": "131",
  "position": "Centre-Forward",
  "maxItems": 40
}
```

### Example: every club in a competition

```json
{
  "mode": "byCompetitionId",
  "competitionId": "GB1",
  "maxItems": 20
}
```

## Use cases

- Scouting reports that need a player's transfer history and current market value in one record.
- Tracking a player's market value trend over their career.
- Building a squad roster for a club, filtered to a position or nationality.
- Comparing total squad value and average age across clubs in the same competition.
- Research on transfer fees and club spending by season.

## FAQ

**Is this affiliated with Transfermarkt?**
No. This actor reads Transfermarkt's public player, club, and competition data; it isn't operated by or connected to Transfermarkt.

**Do I need to log in or provide an API key?**
No. Every mode works on public data with no account required.

**Where do I find a player, club, or competition ID?**
Run `searchPlayers`, `searchClubs`, or `searchCompetitions` first — each result record includes the numeric ID (or code, for competitions) needed for the `byPlayerId`, `byClubId`, or `byCompetitionId` modes. IDs also appear in a Transfermarkt profile URL, e.g. `28003` in `.../profil/spieler/28003`.

**Why do `transfers` and `injuries` only appear with `byPlayerId`?**
Search results return a summary profile; the full transfer and injury history is only fetched when you request a specific player by ID.

**Why is `highestMarketValue` sometimes missing even for a well-known player?**
It's only populated when Transfermarkt's market value history for that player has a valid value to compare; some players have too short a history for a peak to be determined.

**Do the `minMarketValueEur`, `nationality`, and `position` filters apply to clubs and competitions too?**
No, they only filter player records.

**How current is the data?**
Every run queries Transfermarkt live, so market values, squads, and transfer records reflect what the site shows at run time.
