# Google Maps Restaurant Menu & Popular Dish Extractor

Google Maps shows a Menu tab on many restaurant listings with dish names, photos and a "Popular" badge — this actor turns that into structured rows for a single place, along with its menu section names and an external menu link when the business publishes one.

## What this actor does

- Reads the inline menu cards Google renders on a restaurant's place page: item name, photo, Popular flag, and the menu section (e.g. Appetizers, Ramen, Desserts) it belongs to
- Surfaces an external menu link (restaurant site, PDF, or ordering portal) when the listing has one
- Accepts a place by Google Maps URL or by canonical Place ID
- Returns a truthful place-level summary row instead of fake items when Google doesn't serve any inline dish cards for that run — this is common and expected, not a failure
- Empty or unavailable fields are left out of each record rather than sent as placeholders

## Output per record

Google decides, per run, whether it serves inline dish cards for a place — not the input you provide — so a run can return either shape below for the same place on different attempts.

### Item row (`recordType: "menuItem"`) — one per dish

- `businessName` — restaurant name
- `placeId` — canonical Google Place ID (`ChIJ…`), when observed
- `featureId` — legacy feature token (`0x…:0x…`), when `placeId` isn't available
- `placeUrl` — the Google Maps URL for the place
- `menuUrl` — external menu link, when the listing has one
- `menuCategories[]` — the menu section names Google exposes for this restaurant
- `menuFound: true`
- `menuSurfaceRendered: true` — the dish card itself is proof the menu surface rendered
- `itemName` — the dish name
- `isPopular` — `true` when Google flags the item with a Popular badge
- `itemImage` — max-resolution photo of the dish, when available
- `rank` — the item's 1-based position in the menu
- `sourceUrl`, `scrapedAt`

### Summary row (`recordType: "menuSummary"`) — one per place, when no dish cards were served

- The same identity fields as above (`businessName`, `placeId`/`featureId`, `placeUrl`, `menuUrl`, `menuCategories[]`)
- `menuFound: false` — no inline dish cards were served on this run
- `menuSurfaceRendered` — `true` if the Menu panel rendered but held no cards; `false` if the menu surface itself was never observed for this place
- `itemCount: 0`
- `sourceUrl`, `scrapedAt`

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `placeUrl` | string | – | Google Maps URL of the restaurant to scrape (one of `placeUrl` / `placeId`) |
| `placeId` | string | – | Canonical Google Place ID beginning with `ChIJ`, used instead of `placeUrl` |
| `maxItems` | integer | `100` | Maximum menu items to extract (1–100,000); Google may expose fewer |
| `language` | string | `en` | Google Maps UI language: `en`, `es`, `fr`, `de`, `it`, `pt`, `ja`, `ko`, `zh`, `ar`, `ru` |
| `proxyConfiguration` | object | – | Optional proxy; the actor works reliably without one by default |

### Example: menu by place URL

```json
{
  "placeUrl": "https://www.google.com/maps/place/McDonald's/@40.7577297,-73.9853627,17z/",
  "maxItems": 50,
  "language": "en"
}
```

### Example: menu by canonical Place ID

```json
{
  "placeId": "ChIJabtoT7hlwokRLECK0SW3_8I",
  "maxItems": 1000
}
```

### Example: menu in another UI language

```json
{
  "placeUrl": "https://www.google.com/maps/place/Sushi+no+Midori/@35.6704396,139.7596577,17z/",
  "maxItems": 100,
  "language": "ja"
}
```

## Use cases

- **Delivery and food-tech platforms** — populate a dish catalogue with photos and popularity signals
- **Menu intelligence** — track which dishes a restaurant highlights as Popular over time
- **Cuisine and category research** — aggregate menu section names across many restaurants in a cuisine or chain
- **Lead enrichment** — attach menu signals and an external ordering link to a restaurant record
- **Chain monitoring** — compare menu section coverage across branches of the same restaurant brand

## FAQ

**Does every restaurant on Google Maps have a menu?**
Not every one has inline dish cards indexed, and even where they exist Google doesn't serve them on every run — the same place can return cards on one run and none on the next. When no cards are served, the actor returns a `menuSummary` row with `menuFound: false` and any external menu link it did observe, rather than inventing items.

**Why don't items have prices?**
Google Maps' Menu tab shows dish name, photo and a Popular badge, but not per-item prices. When the restaurant publishes full pricing, it lives behind the external link in `menuUrl`.

**What's the difference between `menuFound` and `menuSurfaceRendered`?**
`menuFound` tells you whether at least one dish card was extracted. `menuSurfaceRendered` tells you whether the Menu panel itself was observed to render at all — on a summary row, `true` means the panel rendered but was empty, while `false` means the Menu surface was never seen for that place on this run.

**Why do I sometimes see menu item names in an unexpected language?**
Menu content on Google Maps is aggregated from multiple sources, so a single restaurant's menu can legitimately mix languages (e.g. an English-language US listing showing both "French Fries" and a translated or original-language variant). The actor returns item names exactly as Google published them.

**Can I scrape menus for multiple restaurants in one run?**
This actor processes one `placeUrl` or `placeId` per run. To cover many restaurants, run it once per place — for example from a schedule or a workflow that iterates over a list of URLs.

**How many items can one run return?**
Up to `maxItems` (max 100,000), but the actual count depends entirely on how many dish cards Google serves for that place on that run — anywhere from zero to a few dozen is typical, even for well-known chains.

**Do I need a proxy or login?**
No. The actor reads the public place page directly; no account, cookies, or proxy are required for typical use.

**Is this affiliated with Google?**
No. This is an independent, third-party actor that reads Google Maps' public place pages; it is not affiliated with or endorsed by Google.
