# WTA Singles, Doubles & Race Rankings Finder

Pull the official WTA world rankings table straight from wtatennis.com — singles, doubles, or the Race to the WTA Finals standings — as one structured record per ranked player, with points, nationality, and a link to her profile page.

## What this actor does

- **Three ranking tables:** `singles`, `doubles`, or `race` (Race to the WTA Finals)
- **Name filtering:** narrow the table down to players whose name matches a search term
- **Player identity fields:** WTA player ID, nationality, age, birth date, and profile photo where published
- **Ranking context on every row:** which ranking table (`rankingType`) the row came from
- **Empty fields are omitted** from each record — a player missing a birth date, for example, simply won't have that key

## Output per player

- `ranking` — position in the selected ranking table
- `playerId` — WTA's internal player ID
- `playerName` — player's full name
- `nationality` — country code (e.g. `POL`, `USA`)
- `points` — ranking points
- `age` — player's age, when published
- `birthDate` — date of birth, when published
- `imageUrl` — player photo URL, when published
- `profileUrl` — link to the player's wtatennis.com profile page
- `rankingType` — the ranking table this row was read from: `singles`, `doubles`, or `race`
- `recordType` — always `"wta_player"`
- `scrapedAt` — ISO 8601 timestamp of when the record was collected

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `rankings` | `rankings` or `players`; both read the same ranking table, `players` is intended for use with `searchQuery` |
| `rankingType` | string | `singles` | `singles`, `doubles`, or `race` |
| `searchQuery` | string | – | Only return players whose name contains this text (case-insensitive) |
| `maxItems` | integer | `50` | Maximum player records to return (1–500) |

### Example: top of the singles rankings

```json
{
    "mode": "rankings",
    "rankingType": "singles",
    "maxItems": 100
}
```

### Example: find a specific player's current doubles ranking

```json
{
    "mode": "players",
    "rankingType": "doubles",
    "searchQuery": "Swiatek"
}
```

### Example: Race to the WTA Finals standings

```json
{
    "mode": "rankings",
    "rankingType": "race",
    "maxItems": 20
}
```

## Use cases

- **Tour tracking** — pull the current top-N singles or doubles rankings on a schedule to watch movement week over week
- **Player dashboards** — look up a specific player's ranking and points by name with `searchQuery`
- **Race-to-Finals coverage** — follow qualification standings for the year-end championships as the season progresses
- **Betting and fantasy research** — feed current rankings and points into models that need up-to-date standings
- **Media and content** — source accurate rankings tables for articles or graphics without manual copy-pasting

## FAQ

**Where does this data come from?**
The official WTA rankings as published on wtatennis.com, read from WTA's own rankings data feed.

**What's the actual difference between `mode=rankings` and `mode=players`?**
Both read the same ranking table; `players` is just the intended mode when you're also setting `searchQuery` to find one player, but `searchQuery` works the same way under either mode.

**Do I need a WTA account or API key?**
No. WTA's rankings pages are public; no login or key is required.

**What does `rankingType=race` show?**
The Race to the WTA Finals standings — the season-long points race that determines qualification for the year-end championships, separate from the regular singles/doubles rankings.

**Why might `age`, `birthDate`, or `imageUrl` be missing for a player?**
WTA doesn't publish every biographical field for every ranked player; fields that aren't available are left out of the record instead of appearing blank.

**How current are the rankings?**
As current as wtatennis.com's own published rankings, which WTA updates on its regular weekly ranking cycle.

**Can I pull more than one ranking type in a single run?**
No, one run reads one `rankingType`. Run the actor again with a different `rankingType` to get another table.

**Is this affiliated with the WTA?**
No. This is an independent, third-party actor that reads WTA's publicly published rankings data; it is not operated by the Women's Tennis Association.
