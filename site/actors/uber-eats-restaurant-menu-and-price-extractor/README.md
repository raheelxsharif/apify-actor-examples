# Uber Eats Restaurant Menu & Price Extractor

Give this actor a handful of Uber Eats restaurant page links and it pulls back the full menu — every section, every item, prices and photos — plus the restaurant's own rating, cuisines, delivery fee and estimated delivery time.

## What this actor does

- **Full menu extraction** — every section and item on a restaurant's page, not just a preview.
- **Restaurant profile** — rating, rating count, price tier, cuisines, open status, address and city alongside the menu.
- **Delivery details** — delivery fee and estimated delivery time as shown on the page.
- **Menu completeness stats** — section and item counts, plus how many items actually have a photo or description.
- Fields a restaurant doesn't publish (e.g. no item description, no delivery fee data) are omitted rather than returned empty.

## Output per restaurant

- `restaurantId`, `restaurantName`, `restaurantUrl`
- `heroImageUrl`
- `rating`, `ratingCount`
- `priceRange` — e.g. `$`, `$$`, `$$$`
- `cuisines` — e.g. `["Burgers", "Fast Food"]`
- `isOpen`
- `address`, `city`
- `deliveryFee` — e.g. `Free`, `$0.49`
- `estimatedDeliveryTime` — e.g. `15–25 min`
- `menu` — array of `{sectionName, items[]}`
- `menuSectionCount`, `menuItemCount` — totals across the whole menu
- `itemsWithImageCount`, `itemsWithDescCount` — how many items have a photo or description
- `scrapedAt`

Each entry inside `menu[].items[]` has:

- `itemId`, `name`, `description`
- `price`, `currency`
- `imageUrl`
- `isAvailable`

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `restaurantUrls` | array | – | **Required.** Uber Eats restaurant page URLs to scrape |
| `maxItemsPerRestaurant` | integer | `200` | Maximum menu items to extract per restaurant (1–2000) |
| `country` | string | `us` | `us`, `gb`, `au`, or `ca` — country context for the restaurant's delivery location |

### Example: a single restaurant, full menu

```json
{
  "restaurantUrls": [
    "https://www.ubereats.com/store/mcdonalds-fillmore/P21H_Lf3Se2wSmjGHfoFcQ"
  ],
  "country": "us"
}
```

### Example: several restaurants, capped menu size

```json
{
  "restaurantUrls": [
    "https://www.ubereats.com/store/mcdonalds-fillmore/P21H_Lf3Se2wSmjGHfoFcQ",
    "https://www.ubereats.com/store/some-other-restaurant/AbCdEfGhIjKlMnOp"
  ],
  "maxItemsPerRestaurant": 50
}
```

### Example: a UK restaurant

```json
{
  "restaurantUrls": [
    "https://www.ubereats.com/gb/store/some-london-restaurant/QrStUvWxYzAbCdEf"
  ],
  "country": "gb"
}
```

## Use cases

- **Competitive intelligence** — compare a competitor's prices and menu composition against your own.
- **Menu database building** — collect structured menus across many restaurants in one run.
- **Price monitoring** — rerun the same URLs on a schedule to track price changes over time.
- **Market analysis** — look at cuisine and pricing patterns across a city or chain.
- **Catalog migration** — import a restaurant's menu structure into another ordering or review platform.

## FAQ

**Which countries are supported?**
`us`, `gb`, `au`, and `ca`. Pick the one matching the restaurant's physical location for the most accurate delivery details.

**Can I scrape several restaurants in one run?**
Yes — list as many URLs as you want in `restaurantUrls`; each produces its own output record.

**Where do I get a restaurant's URL?**
Open the restaurant on ubereats.com and copy the page URL from your browser — it looks like `https://www.ubereats.com/store/<name>/<id>`.

**Why do some items have no `description`?**
Many Uber Eats restaurants simply don't write descriptions for their menu items. `itemsWithDescCount` tells you how many items on that menu actually have one; photos (`itemsWithImageCount`) are far more commonly present.

**How fresh is the data?**
Each run fetches the menu live from Uber Eats at request time.

**Does this need a login or proxy?**
No — restaurant menu pages are public, so no login, API key, or proxy is required.

**Is this affiliated with Uber Eats?**
No. This is an independent, third-party actor that reads Uber Eats' publicly available restaurant pages.
