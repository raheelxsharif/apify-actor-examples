# OP.GG League of Legends Tier List & Win Rate Tracker

Read live **OP.GG League of Legends** tier lists and champion statistics without an account, API key, or proxy. Pick a server, rank, queue, lane, and patch to get champion tiers with win/pick/ban rates, role share, rank movement, and the three strongest counters — or switch to champion statistics for games played, KDA, CS, and gold.

## What this actor does

- **Three modes:** `tierList` for OP.GG's ranked champion table, `championStats` for per-champion performance metrics, and `fullCounters` for every matchup of selected champions.
- **Precise scope:** Global plus 15 servers, 16 player ranks, Ranked Solo/Duo, Ranked Flex, Classic, ARAM statistics, and all five lanes.
- **Patch-aware:** read the latest patch or the previous one; Classic returns OP.GG's current Classic table.
- **Rich tier rows:** current OP.GG tier (OP, 1–5), position rank, rank change since yesterday and since the previous patch, win/pick/ban/role rates, and top-three counters.
- **Champion statistics:** games played, KDA, kills/deaths/assists, average CS, average gold, and win/pick/ban rates for a month, week, or today.
- **Filter by champion tier:** keep only OP, 1 tier, or any cutoff down to 5 tier.
- **Clean output:** empty fields are omitted, so no `null` or blank values appear in the dataset.

## Output per champion tier row (mode = `tierList`)

- `championName`, `championKey`, `championId` — champion identity.
- `position` — lane the row applies to (`TOP`, `JUNGLE`, `MID`, `ADC`, `SUPPORT`).
- `tier` — current OP.GG tier label: `OP`, `1`, `2`, `3`, `4`, or `5`.
- `tierNumber` — numeric tier (`0` = OP, `1`–`5`).
- `tierRank` — champion's rank within its lane.
- `rankPrev`, `rankPrevPatch` — rank on the previous day and previous patch.
- `rankChange` — places gained (positive) or lost (negative) since yesterday.
- `winRate`, `pickRate`, `banRate` — percentages for the selected scope.
- `roleRate` — share of the champion's games played in this lane.
- `weakAgainst[]` — top three counters, each with `championName`, `championKey`, `championId`, `games`, `wins`, and `winRate`.
- `roles[]` — OP.GG class labels for Classic rows.
- `patch` — patch version behind the statistics.
- `imageUrl`, `championUrl` — champion portrait and OP.GG build page.
- `region`, `rankTier`, `queueType` — the filters that produced the row.
- `recordType` (`champion`), `sourceUrl`, `scrapedAt`.

## Output per champion stat row (mode = `championStats`)

- `championName`, `championKey` — champion identity.
- `position` — selected lane, or `ALL`.
- `gamesPlayed` — sample size in the selected period.
- `kda` — average kills + assists divided by deaths.
- `kills`, `deaths`, `assists` — totals across the sampled games.
- `averageCs`, `averageGold` — per-game averages.
- `winRate`, `pickRate`, `banRate` — percentages.
- Rows are ordered by games played, most-played first.
- `queueType`, `region`, `rankTier`, `period` — the filters that produced the row.
- `patch`, `championUrl`, `imageUrl`, `recordType` (`championStats`), `sourceUrl`, `scrapedAt`.

## Output per counter row (mode = `fullCounters`)

- `targetChampionName`, `targetChampionKey`, `targetChampionId` — the champion you asked for.
- `championName`, `championKey`, `championId` — the opponent.
- `position` — lane the matchup belongs to.
- `games`, `wins` — games in the matchup and the target champion's wins.
- `winRate` — target champion's win rate against that opponent.
- `region`, `rankTier`, `queueType`, `patch` — the filters that produced the row.
- `recordType` (`counter`), `sourceUrl`, `scrapedAt`.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `tierList` | `tierList` for champion tiers and counters, `championStats` for games/KDA/CS/gold; `byPosition` is a legacy alias for a position-scoped tier list. |
| `queueType` | string | `ranked` | `ranked`, `flex`, `classic` (tier list only), or `aram` (champion stats only). |
| `region` | string | `global` | `global`, `na`, `me`, `euw`, `eune`, `oce`, `kr`, `jp`, `br`, `las`, `lan`, `ru`, `tr`, `sea`, `tw`, `vn`. |
| `rankTier` | string | `emerald_plus` | `all`, `challenger`, `grandmaster`, `master_plus`, `master`, `diamond_plus`, `diamond`, `emerald_plus`, `emerald`, `platinum_plus`, `platinum`, `gold_plus`, `gold`, `silver`, `bronze`, `iron`. |
| `position` | string | `ALL` | `ALL`, `TOP`, `JUNGLE`, `MID`, `ADC`, `SUPPORT`. In `fullCounters`, `ALL` uses each champion's primary lane. |
| `champions` | array | – | Champion names, keys, or OP.GG build URLs to fetch every matchup for (mode=fullCounters), e.g. `["Ahri", "jinx", "K'Sante"]`. |
| `patch` | string | `latest` | `latest` or `previous` (mode=tierList or fullCounters). |
| `minTier` | string | `all` | Keep champions at or above this tier: `all`, `0` (OP), `1`–`5`. |
| `period` | string | `month` | `month`, `week`, or `day` (mode=championStats). |
| `maxItems` | integer | `100` | Hard cap on records (1–1000). |

### Example: default ranked tier list

```json
{
  "mode": "tierList",
  "queueType": "ranked",
  "region": "global",
  "rankTier": "emerald_plus",
  "position": "ALL",
  "patch": "latest",
  "minTier": "all",
  "maxItems": 100
}
```

### Example: Korea Challenger mid lane, previous patch

```json
{
  "mode": "tierList",
  "queueType": "ranked",
  "region": "kr",
  "rankTier": "challenger",
  "position": "MID",
  "patch": "previous",
  "minTier": "2",
  "maxItems": 50
}
```

### Example: Classic support tier list

```json
{
  "mode": "tierList",
  "queueType": "classic",
  "region": "global",
  "rankTier": "emerald_plus",
  "position": "SUPPORT",
  "minTier": "all",
  "maxItems": 30
}
```

### Example: ARAM champion statistics for the last week

```json
{
  "mode": "championStats",
  "queueType": "aram",
  "region": "global",
  "rankTier": "all",
  "position": "ALL",
  "period": "week",
  "maxItems": 50
}
```

### Example: every matchup for Ahri and Jinx

```json
{
  "mode": "fullCounters",
  "queueType": "ranked",
  "region": "kr",
  "rankTier": "diamond_plus",
  "position": "MID",
  "champions": ["Ahri", "https://op.gg/lol/champions/jinx/build/adc"],
  "maxItems": 120
}
```

## Use cases

- **Draft preparation** — pull the strongest picks for a lane and rank before a ranked session.
- **Meta tracking** — compare the latest and previous patch, or track daily rank movement.
- **Content creation** — build tier-list graphics with win/pick/ban rates and counters.
- **Champion research** — inspect KDA, CS, and gold for a champion across servers and ranks.
- **Coaching tools** — feed current counters and role rates into pick recommendations.
- **Esports analysis** — compare regional metas and lane priorities.

## FAQ

### Do I need an OP.GG account, API key, or proxy?

No. The actor reads OP.GG's public pages and returns data without any credentials or proxy setup.

### What do the tier labels mean?

OP.GG now labels the strongest champions `OP`, followed by `1`, `2`, `3`, `4`, and `5`. Lower numbers are stronger. The older `OP+`, `S+`, and `A+` labels are no longer used by OP.GG. `tierNumber` is the tier itself; `tierRank` is the champion's rank within its lane (for example, the 4th-best mid laner).

### Why is `weakAgainst` missing from some rows?

OP.GG omits counters when the matchup sample is too small. This is most common on Challenger, Grandmaster, and smaller servers. The rest of the row is still returned. More generally, every field is omitted when OP.GG does not provide it for that row; the actor never emits `null`, empty strings, or empty arrays.

### Why does champion stats use Emerald instead of Emerald+?

OP.GG's statistics page only supports exact ranks (Emerald, Diamond, Master, and so on), not “plus” ranks. The actor automatically uses the base rank and reports the effective rank in its status message.

### What is the difference between `tierList`, `championStats`, and `fullCounters`?

`tierList` returns one row per champion and lane with OP.GG's tier, win/pick/ban rates, rank movement, and top-three counters. `championStats` returns one row per champion with games played, KDA, kills/deaths/assists, CS, and gold for the selected queue, region, rank, and period. `fullCounters` returns every matchup for the champions you list, one row per opponent.

### How do I get every matchup instead of just the top three counters?

Use `mode=fullCounters` and list champions in `champions`. Each champion is read from OP.GG's counters page for the selected lane, queue, region, rank, and patch. OP.GG does not publish ARAM counters; ranked, flex, and classic are supported. Increase `maxItems` to keep more matchups.

### Can I get ARAM or Classic?

Yes. Use `mode=championStats` with `queueType=aram` for ARAM statistics. Classic is available in `tierList` and `fullCounters`; OP.GG's Classic data is a single global pool and has no patch history, so region and player-rank filters do not change it.

### Does this actor return runes, items, or skill order?

No. This actor covers tier lists, champion statistics, and full matchup counters. OP.GG's champion build pages and its official champion-analysis tool provide runes, items, and skill order; those are outside this actor's scope.

### How current is the data?

OP.GG recalculates its tables continuously from recent matches. The actor reads the live page each run; `patch` shows which game patch the numbers came from. OP.GG keeps only the latest and previous patch, so older patches are not available.

### Is this affiliated with OP.GG or Riot Games?

No. This is a third-party actor that reads OP.GG's publicly available statistics pages. It is not affiliated with, endorsed by, or sponsored by OP.GG or Riot Games.
