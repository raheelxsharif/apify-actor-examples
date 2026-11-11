# Google Maps Restaurant Menu & Price Scraper

Scrape restaurant menus from **Google Maps** and the restaurant's own public menu page. Paste one or many place URLs, or search by cuisine and city, and get one row per dish with its section, description, price, currency, photo, menu provenance, and the full place context — address, rating, review count, categories, website and coordinates. No login, API key or proxy setup is required.

## What this actor does

- **Two modes:** `byPlace` (Maps URL, short link, Place ID or feature ID) and `bySearch` (cuisine + city query)
- **Batch-ready:** process many restaurants in one run, with a separate item cap per place
- **Dish-level data:** menu section, item name, description, price text, numeric price and currency
- **Menu provenance:** `menuSource`, `menuProvider`, the menu URL and the sections found for each restaurant
- **Place context on every row:** address, rating, review count, price level, categories, website, latitude and longitude
- **Honest no-menu rows:** when a place has no readable menu, one summary row records the place instead of inventing items
- Empty or unavailable fields are omitted from each row rather than sent as `null`

## Output per menu item (`recordType: "menuItem"`)

- `businessName`, `placeId`, `featureId`, `placeUrl`
- `placeAddress`, `placeRating`, `placeReviewCount`, `placePriceLevel`
- `placeCategories[]`, `placeWebsite`, `latitude`, `longitude`
- `menuUrl`, `menuCategories[]` — the menu page and the sections detected on it
- `menuSource` — `menuPage` when the restaurant's own menu supplied the items, `google` when Google's inline menu did
- `menuProvider` — the menu site or provider label, when identifiable
- `menuSection` — the dish's section, e.g. `Antipasti` or `Desserts`
- `itemName`, `itemDescription`
- `priceText` — the price exactly as published, e.g. `$24.95`, `€12,50`, `12.00 USD`
- `price` — numeric value when the price is unambiguous
- `currency` — ISO code when the page states it or the symbol is unambiguous
- `itemImage` — dish photo when the menu publishes one
- `isPopular` — `true` only when Google's inline menu shows a Popular badge
- `rank` — 1-based position in the collected menu
- `totalItems` — rows returned for this restaurant
- `menuFound: true`, `recordType: "menuItem"`, `sourceUrl`, `scrapedAt`

## Output per place without a readable menu (`recordType: "menuSummary"`)

- The same place context fields and `menuUrl` when one was found
- `menuFound: false`, `menuSource: "none"`
- `itemCount: 0`, `totalItems: 0`
- `recordType: "menuSummary"`, `sourceUrl`, `scrapedAt`

When `includePopularDishes` is enabled and Google exposes Popular badges, an additional summary row carries `popularDishes[]` beside the item rows.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `byPlace` | `byPlace` or `bySearch` |
| `placeUrls` | array | – | Maps URLs, `maps.app.goo.gl` links, `ChIJ…` Place IDs or `0x…:0x…` feature IDs (mode=byPlace) |
| `searchQueries` | array | – | Cuisine + city queries, e.g. `italian restaurant, New York` (mode=bySearch) |
| `maxPlacesPerQuery` | integer | `20` | Restaurants to process per search query (1–120, mode=bySearch) |
| `maxItemsPerPlace` | integer | `100` | Maximum menu rows per restaurant (1–1000) |
| `language` | string | `en` | Language for Maps lookups and search results |
| `includePlacesWithoutMenu` | boolean | `true` | Return a summary row for places with no readable menu |
| `includePopularDishes` | boolean | `false` | Also check Google's inline menu for Popular badges (slower) |

### Example: several restaurants by URL

```json
{
  "mode": "byPlace",
  "placeUrls": [
    "https://www.google.com/maps/place/Sicily+Osteria/@40.7601601,-73.9891863,17z/data=!3m1!4b1!4m6!3m5!1s0x89c25932322b79d3:0x7c085aeae6751161!8m2!3d40.7601601!4d-73.9891863",
    "https://www.google.com/maps/place/?q=place_id:ChIJxZXYx7cEdkgRdgAOZ6OHOJw"
  ],
  "maxItemsPerPlace": 50
}
```

### Example: search by cuisine and city

```json
{
  "mode": "bySearch",
  "searchQueries": ["italian restaurant, New York", "sushi in Austin"],
  "maxPlacesPerQuery": 10,
  "maxItemsPerPlace": 25,
  "includePlacesWithoutMenu": true
}
```

### Example: place IDs only

```json
{
  "mode": "byPlace",
  "placeUrls": ["ChIJ03krMjJZwokRYRF15upaCHw"],
  "maxItemsPerPlace": 100,
  "language": "en"
}
```

### Example: non-English menu lookup

```json
{
  "mode": "byPlace",
  "placeUrls": ["https://www.google.com/maps/place/?q=place_id:ChIJxZXYx7cEdkgRdgAOZ6OHOJw"],
  "language": "de",
  "maxItemsPerPlace": 30
}
```

## Use cases

- **Menu intelligence** — track dish names, sections and prices across a city or chain
- **Delivery and food-tech catalogues** — seed dish data with descriptions and price text
- **Restaurant lead enrichment** — attach menu coverage, website and rating to a place list
- **Market research** — compare cuisine pricing and menu depth across neighbourhoods
- **Hospitality monitoring** — detect menu changes, new sections and removed dishes over time

## FAQ

**Where does the menu data come from?**
From Google Maps place data plus the restaurant's own public menu page when it publishes one. The actor reads the menu items the restaurant publishes, not an AI guess.

**Why is there no menu for some restaurants?**
Not every restaurant publishes a structured menu. Coverage in tested cities is roughly 60–80%; places that publish only a PDF, an image, or a login-gated ordering page may not expose readable item rows. When `includePlacesWithoutMenu` is on, those places still produce one summary row so nothing is silently dropped.

**Why do some items have no price or no description?**
Many menus publish dish names without prices, or prices without descriptions. The actor reports what the page actually contains. `priceText` preserves the published price even when it cannot be interpreted as a single number, and `price`/`currency` are omitted when the value is ambiguous (for example a price range).

**What is the difference between `menuSource` and `menuProvider`?**
`menuSource` tells you whether the rows came from `menuPage` or Google's inline menu (`google`). `menuProvider` names the menu site or platform when it can be identified, which helps when you compare the same chain across locations.

**What happens when `maxItemsPerPlace` is smaller than the published menu?**
Rows are returned in menu order up to the cap, and `totalItems` states how many rows this run returned for that place. Increase the cap to collect more.

**Can I scrape many restaurants in one run?**
Yes. Use several `placeUrls` entries in `byPlace`, or one or more `searchQueries` in `bySearch`. Places are deduplicated by Place ID or feature ID before menus are fetched.

**Do I need a proxy or a Google API key?**
No. The actor works without a key, login or proxy setup. It automatically chooses a working connection and only escalates when a source blocks the direct request.

**How fresh is the data?**
Each run reads the current public menu and place data; nothing is cached between runs. Menu changes appear on the next execution.

**Is this affiliated with Google?**
No. This is an independent third-party actor that reads public Google Maps place data and public restaurant menu pages.
