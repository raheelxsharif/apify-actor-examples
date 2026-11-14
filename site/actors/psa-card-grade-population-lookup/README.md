# PSA Card Grade Population Lookup

PSA's population report is the reference database for how many of a given trading card have been certified at each grade. This actor looks it up by keyword search, by PSA spec number, by browsing a full set, or by pulling recently graded cards — returning the card's identity alongside its grade 1–10 population counts.

## What this actor does

- Searches PSA's population database by free-text keyword
- Fetches population data directly for one or more known PSA spec numbers
- Lists every card in a given PSA set by set ID
- Pulls recently graded cards
- Filters by sport/category across 10 options including baseball, basketball, Pokemon and non-sports
- Fields PSA doesn't publish for a card are left out of the record rather than sent as `null`

## Output per card

- `specNumber` — PSA's spec number uniquely identifying this card variety
- `cardNumber` — the card's number within its set
- `cardName` — card title/description
- `year` — card year
- `setName` — the set the card belongs to
- `sport` — sport or category (e.g. baseball, Pokemon, non-sports)
- `brand` — publisher/brand
- `variety` — card variety or variation, when the card has one
- `subject` — the featured player or subject
- `grade1Count` through `grade10Count` — population at each PSA grade
- `totalGraded` — total number of copies graded across all grades
- `imageUrl` — card image, when available
- `popUrl` — link to this card's PSA population report page
- `recordType: "card"`, `scrapedAt`

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `searchCards` | `searchCards` / `cardPopulation` / `recentGrades` / `searchBySet` |
| `query` | string | `1952 Topps Mickey Mantle` | Search keyword (mode=searchCards) |
| `specNumbers` | array | – | PSA spec numbers to fetch population for (mode=cardPopulation) |
| `setId` | integer | – | PSA set ID to list cards from (mode=searchBySet) |
| `sport` | string | – (any) | `baseball` / `basketball` / `football` / `hockey` / `soccer` / `pokemon` / `non-sports` / `golf` / `tennis` / `boxing` |
| `maxItems` | integer | `20` | Maximum number of cards to return (1–200) |

### Example: search by card name

```json
{
  "mode": "searchCards",
  "query": "1952 Topps Mickey Mantle",
  "maxItems": 20
}
```

### Example: population for known spec numbers

```json
{
  "mode": "cardPopulation",
  "specNumbers": ["84233", "12345"]
}
```

### Example: browse a full set

```json
{
  "mode": "searchBySet",
  "setId": 123456,
  "maxItems": 100
}
```

### Example: recently graded Pokemon cards

```json
{
  "mode": "recentGrades",
  "sport": "pokemon",
  "maxItems": 50
}
```

## Use cases

- **Collectors** — check how rare a specific grade is for a card before buying or selling
- **Grading strategy** — compare population across grades to decide whether resubmitting a card for a higher grade is worthwhile
- **Marketplace pricing** — combine population counts with sale data to gauge scarcity-driven value
- **Set tracking** — pull every card in a set to see which ones are hardest to find in top grades
- **Market monitoring** — watch recently graded cards in a category to spot fresh submissions

## FAQ

**Does this require a PSA account?**
No. It reads PSA's public population report pages; no login is required.

**What is a PSA spec number?**
A spec number is PSA's unique identifier for a specific card variety (set, year, card number and variation combined), distinct from the generic card number printed on the card itself.

**What does the population count actually mean?**
It's the running total of how many submitted copies of that card PSA has certified at each grade level, from 1 up to the top grade of 10, as of PSA's current data.

**Which sports and categories can I filter by?**
Baseball, basketball, football, hockey, soccer, Pokemon, non-sports, golf, tennis, and boxing.

**Why are some grade fields missing from a card's record?**
A grade count field only appears when PSA reports at least one card graded at that level; a grade with zero submissions isn't included.

**How current is the population data?**
Each run reads PSA's live population report, so counts reflect PSA's most recently published data at the time of the run.

**Is this affiliated with PSA (Professional Sports Authenticator)?**
No. This is an independent, third-party actor that reads PSA's publicly accessible population report; it is not affiliated with or endorsed by PSA or Collectors Universe.
