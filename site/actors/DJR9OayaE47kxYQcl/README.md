# eBay Motors Search & Listing Detail Extractor

eBay Motors carries hundreds of thousands of vehicle and parts listings from private sellers and dealers across the US. This actor runs an eBay Motors search for a vehicle type or parts category, or reads a list of specific item URLs you already have, and turns every listing into a structured record with pricing, vehicle specs, seller details and images.

## What this actor does

- Searches eBay Motors vehicle listings by keyword, vehicle category (cars & trucks, motorcycles, boats, RVs, ATVs, commercial trucks), condition, price range, model-year range and sort order
- Searches eBay Motors parts & accessories listings by keyword and parts category
- Enriches a list of specific eBay item URLs with full listing detail instead of running a search
- Captures auction-specific data (bid count, time remaining) alongside Buy It Now pricing
- Empty fields are omitted from records — a field only appears when the listing actually shows that data

## Output per listing

- `listingId` — eBay item ID
- `title` — full listing title
- `url` — canonical eBay listing URL
- `price` — numeric price in USD
- `priceFormatted` — price as displayed (e.g. `$22,500`)
- `currency` — currency code for `price`
- `originalPrice` — pre-discount price, when the listing shows a markdown
- `condition` — listing condition (e.g. `Used`, `New`)
- `titleStatus` — vehicle title status (e.g. `Clean title`, `Salvage title`), when shown
- `mileage` — vehicle mileage
- `year`, `make`, `model`, `trim` — vehicle identity fields
- `bodyStyle` — e.g. `Sedan`, `SUV`
- `fuelType` — e.g. `Gasoline`, `Electric`
- `transmission` — e.g. `Automatic`, `Manual`
- `drivetrain` — e.g. `FWD`, `AWD`, `4WD`
- `exteriorColor`
- `sellerName`, `sellerUrl` — seller username and profile URL
- `sellerFeedbackScore`, `sellerPositivePercent` — seller reputation stats
- `dealerName`, `dealerFeedbackPercent`, `dealerFeedbackCount` — present when the listing is sold by a dealership rather than a private seller
- `shippingCost`, `shippingType` — shipping cost in USD and type (`Free`, `Calculated`, `Freight`)
- `location` — seller's city/state
- `deliveryOrPickupAvailable` — `true` when delivery or local pickup is offered
- `imageUrl` — primary listing image
- `images` — every image URL on the listing
- `listingType` — `BuyItNow`, `Auction` or `BestOffer`
- `timeRemaining`, `bids` — auction countdown text and bid count (auction listings)
- `watchCount`, `viewCount`
- `vehicleType` — vehicle category searched (e.g. `cars-trucks`)
- `partType` — parts category, present on parts listings
- `isTrendingRecommendation` — present and `true` only when `searchVehicles` runs with no `searchQuery` and no price/year/condition filters, meaning eBay served its curated "Trending Vehicles" module rather than real filtered search matches; omitted entirely on normal searches
- `recordType` — always `ebayMotorsListing`
- `scrapedAt` — ISO 8601 UTC timestamp

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `searchVehicles` | `searchVehicles`, `searchParts`, or `byListingUrls` to enrich specific item URLs |
| `searchQuery` | string | – | Keyword(s), e.g. `honda civic`. Leave empty in `searchVehicles` mode to browse eBay's trending vehicles for `vehicleType` instead (see `isTrendingRecommendation`) |
| `vehicleType` | string | `cars-trucks` | Vehicle category for `searchVehicles` mode |
| `partCategory` | string | `car-truck-parts` | Parts category for `searchParts` mode |
| `condition` | string | `all` | `all`, `new`, `used`, `certified-pre-owned`, or `for-parts-not-working` |
| `sortBy` | string | `best-match` | `best-match`, `price-asc`, `price-desc`, `distance-nearest`, `newly-listed`, or `ending-soonest` |
| `minPrice` | integer | – | Minimum listing price in USD |
| `maxPrice` | integer | – | Maximum listing price in USD |
| `minYear` | integer | – | Minimum model year, for `searchVehicles` mode |
| `maxYear` | integer | – | Maximum model year, for `searchVehicles` mode |
| `startUrls` | array | – | eBay Motors item URLs to scrape directly, for `byListingUrls` mode |
| `maxItems` | integer | `50` | Maximum listings to return (1–5000) |

### Example: used Honda Civics under $15,000

```json
{
  "mode": "searchVehicles",
  "searchQuery": "honda civic",
  "condition": "used",
  "maxPrice": 15000,
  "maxItems": 50
}
```

### Example: motorcycle parts search

```json
{
  "mode": "searchParts",
  "partCategory": "motorcycle-parts",
  "searchQuery": "exhaust system",
  "maxItems": 100
}
```

### Example: enrich specific listing URLs

```json
{
  "mode": "byListingUrls",
  "startUrls": [
    "https://www.ebay.com/itm/256789012345",
    "https://www.ebay.com/itm/198765432109"
  ]
}
```

### Example: 2015–2020 trucks sorted by lowest price

```json
{
  "mode": "searchVehicles",
  "vehicleType": "cars-trucks",
  "minYear": 2015,
  "maxYear": 2020,
  "sortBy": "price-asc",
  "maxItems": 75
}
```

## Use cases

- **Vehicle shopping research** — pull every matching listing for a make/model to compare price, mileage and title status side by side
- **Dealer inventory tracking** — re-run a saved search to see which dealer listings are new, price-changed or sold
- **Parts sourcing** — search a specific parts category and keyword to find available stock and pricing across sellers
- **Auction monitoring** — track `bids`, `timeRemaining` and `watchCount` on listings you're bidding on
- **Listing enrichment** — feed a list of item URLs gathered elsewhere into `byListingUrls` mode to get full structured detail on each

## FAQ

**What's the difference between the three modes?** `searchVehicles` and `searchParts` run an eBay Motors search with your filters and return the matching listings. `byListingUrls` skips search entirely and scrapes exactly the item URLs you supply.

**Why do I get `isTrendingRecommendation: true` records?** That only happens in `searchVehicles` mode when you leave `searchQuery` empty and apply no price/year/condition filters — eBay shows a curated "Trending Vehicles" module instead of a real search results page in that case. Add a keyword or any filter to get genuine matched search results.

**Can I filter parts listings by price or model year?** `minPrice`/`maxPrice` apply to both vehicle and parts searches; `minYear`/`maxYear` only apply to `searchVehicles` mode since most parts listings aren't tied to a single model year.

**Why are `mileage`, `year`, `make` or `model` missing from a parts listing?** Those are vehicle-specific fields. Parts listings instead carry `partType` and whatever specs eBay shows for that part.

**Why might `dealerName` appear but not `sellerName`, or vice versa?** A listing is sold either by a private seller or by a dealership, not both — the actor reports whichever eBay actually shows on that listing.

**Does this include sold or ended listings?** No, only currently active listings are returned.

**Is this affiliated with eBay?** No. This is an independent, third-party actor that reads eBay Motors' public listing pages; it is not operated by or affiliated with eBay Inc.

**How many listings can one run return?** Up to the `maxItems` value you set, capped at 5000; eBay serializes result pages at roughly 25 listings each, so very large runs take proportionally longer.
