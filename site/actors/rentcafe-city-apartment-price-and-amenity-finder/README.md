# RentCafe City Apartment Price & Amenity Finder

RentCafe.com lists apartment communities across the US with per-unit-type pricing, amenities and management company information. This actor searches RentCafe for a city, with an optional bedroom-count, price-tier or amenity filter, and returns each apartment community as a structured record ready for a rent-comparison spreadsheet or a lead list.

## What this actor does

- Searches apartment listings for any US city on RentCafe, identified by its location slug
- Applies one RentCafe site filter at a time — bedroom count, price tier (e.g. under $1,000) or an amenity like pet-friendly, pool or garage
- Refines results further on the client side by minimum rating, a price range, and a pet-friendly-only toggle, stacked on top of the site filter
- Paginates automatically up to your requested item count
- Empty fields are omitted from records — a field only appears when RentCafe actually published that data for the property

## Output per listing

- `propertyId`, `name`, `url` — RentCafe's internal ID, community name and listing URL
- `street`, `city`, `state`, `zip`, `fullAddress`
- `latitude`, `longitude`
- `rating`, `reviewCount` — star rating and number of reviews
- `bedPricePairs` — array of `{beds, startingPrice}` for every unit type advertised
- `minPrice`, `maxPrice` — lowest and highest starting price across unit types
- `minBeds`, `maxBeds` — smallest and largest advertised bedroom count
- `amenitySummary` — list of amenities shown on the listing card
- `badges` — promo tags such as "Rent specials" or "New Units Available"
- `verifiedListing` — `true` when RentCafe marks the listing as verified
- `photoUrl` — listing photo
- `managementCompany` — name of the company managing the property, when shown
- `locationSlug` — the normalized city slug used for the search
- `filterApplied` — the site filter segment used for this run, if any
- `scrapedAt` — UTC timestamp when the record was collected

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `location` | string | – | City as it appears on RentCafe, e.g. `austin-tx`, `Austin, TX`, or `new-york-city-ny`; spaces/commas are converted to the RentCafe URL slug automatically |
| `filter` | string | – (none) | One RentCafe site filter: bedroom count, price tier, or an amenity such as `pet-friendly`, `pool` or `garage` (36 options) |
| `minRating` | integer | – | Drop listings rated below this (0–5) |
| `minPrice` | integer | – | Drop listings whose lowest advertised starting price is below this |
| `maxPrice` | integer | – | Drop listings whose highest advertised starting price is above this |
| `petFriendlyOnly` | boolean | `false` | Only emit listings whose amenity summary mentions "Pet friendly" |
| `maxItems` | integer | `25` | Maximum records to return (1–2000). RentCafe itself typically surfaces at most ~500 listings for a single location/filter combination, even in the largest cities |

### Example: 2-bedroom apartments in Austin, rated 4+, under $2,000/mo

```json
{
  "location": "austin-tx",
  "filter": "2-bedroom",
  "minRating": 4,
  "maxPrice": 2000,
  "maxItems": 50
}
```

### Example: pet-friendly listings in New York City

```json
{
  "location": "new-york-city-ny",
  "filter": "pet-friendly",
  "petFriendlyOnly": true,
  "maxItems": 100
}
```

### Example: luxury apartments in Los Angeles

```json
{
  "location": "los-angeles-ca",
  "filter": "luxury",
  "maxItems": 40
}
```

## Use cases

- **Rent comparison** — pull every listing in a city for a bedroom count and compare starting prices side by side
- **Lease shopping** — filter by amenity (pool, garage, pet-friendly) to shortlist communities that fit a specific need
- **Property management leads** — collect `managementCompany` across a city to build a contact list
- **Market research** — track `minPrice`/`maxPrice` across a city's listings over time to watch rent trends
- **Pet owner search** — combine the `pet-friendly` filter with `petFriendlyOnly` for a narrower, double-checked result set

## FAQ

**How do I find the location slug for my city?** The URL path segment right after `/apartments-for-rent/` on RentCafe.com is the slug to use (e.g. `rentcafe.com/apartments-for-rent/austin-tx/` → `austin-tx`). You can also just type `City, ST` and the actor will normalize it, though results depend on RentCafe recognizing the generated slug.

**Can I combine two site filters, like 2-bedroom and pet-friendly?** RentCafe's search only supports one filter segment per page. Pick the axis that matters most for `filter`, then use `minPrice`/`maxPrice`/`minRating`/`petFriendlyOnly` for additional narrowing on top of it.

**Do I need a login or proxy?** No — RentCafe's listing pages are publicly accessible, and this actor needs no account, API key or proxy.

**Why do some listings have fewer fields than others?** RentCafe only shows what each property manager provided — not every listing has a star rating or amenity list yet. Fields with no real data on the listing are omitted rather than returned empty.

**Why does RentCafe cap results around 500 per location?** That's a limit of RentCafe's own search results, not of this actor; even for the largest cities, a single location/filter combination rarely surfaces more than ~500 listings.

**Is this affiliated with RentCafe?** No. This is an independent, third-party actor that reads RentCafe's publicly available listing pages; it is not operated by or affiliated with RentCafe/Yardi.
