# Apartments.com Rental Listing & Floor Plan Scraper

Apartments.com lists rental properties across the US, from large managed communities down to single-unit houses. This actor searches that inventory by city and filters, or scrapes specific listing pages you already have, and returns each property as a structured record with pricing, floor plans, and contact details.

## What this actor does

- Searches rentals by city/state with filters for bedrooms, price range, property type, pet policy, and amenities.
- Scrapes specific Apartments.com search-result or listing URLs you supply directly.
- Optionally visits each listing's own page for its full amenity list, walkability scores, per-unit floor plans, and leasing contact number.
- Caps both the number of listings returned and the number of search pages crawled.
- Leaves out fields a listing doesn't publish (no coordinates, no rent specials) rather than sending them empty.

## Output per listing

- `listingId`, `url`, `name`
- `address`, `city`, `state`, `zipCode`, `country` — when the listing publishes a country
- `latitude`, `longitude`
- `priceMin`, `priceMax` — monthly rent range across the property's unit types
- `bedsMin`, `bedsMax`, `bathsMin`, `bathsMax`, `sqftMin`, `sqftMax` — ranges across unit types
- `units` — per-floor-plan array of `{rentMin, rentMax, bedsMin, bedsMax, sqftMin, sqftMax}`, from the detail page
- `propertyType`
- `amenities` — list of amenity names, from the detail page
- `description` — property description, from the detail page
- `hasRentSpecials` — present and `true` only when the listing currently advertises a move-in deal
- `availabilityDate`, `leaseTerms` — from the detail page
- `phoneNumber` — leasing office contact number
- `walkScore`, `transitScore`, `bikeScore`, `soundScore` — from the detail page, when published
- `photos` — photo URLs (up to 20 from the detail page, fewer from the search card)
- `scrapedAt` — ISO timestamp of the fetch

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `search` | `search` (build a search from location + filters) or `urls` (scrape the URLs you provide) |
| `searchLocation` | string | – | City and state to search, e.g. `Chicago, IL`; used by `search` |
| `bedrooms` | string | `any` | `any`, `studio`, `1`, `2`, `3`, `4`, or `5plus` |
| `priceMin` | integer | – | Minimum monthly rent in USD |
| `priceMax` | integer | – | Maximum monthly rent in USD |
| `propertyType` | string | `any` | `any`, `apartments`, `houses`, `condos`, or `townhomes` |
| `petPolicy` | string | `any` | `any`, `dogs`, `cats`, or `pets` |
| `amenities` | array | `[]` | Any of `pool`, `utilities-included`, `garage`, `air-conditioning`, `dishwasher`, `walk-in-closets`, `balcony`, `furnished` |
| `rentSpecials` | boolean | `false` | Only return listings currently advertising a move-in deal |
| `startUrls` | array | – | Apartments.com search-result or listing URLs; used by `urls` |
| `scrapeDetails` | boolean | `true` | Visit each listing's own page for full amenities, scores, floor plans, and contact info |
| `maxItems` | integer | `50` | Maximum number of listings to return (1-500) |
| `maxPages` | integer | `5` | Maximum number of search result pages to crawl (1-20); used by `search` |

### Example: pet-friendly two-bedroom search

```json
{
  "mode": "search",
  "searchLocation": "Austin, TX",
  "bedrooms": "2",
  "priceMax": 2500,
  "petPolicy": "dogs",
  "amenities": ["pool", "garage"],
  "maxItems": 100,
  "scrapeDetails": true
}
```

### Example: scrape listing URLs directly

```json
{
  "mode": "urls",
  "startUrls": [
    "https://www.apartments.com/the-domain-austin-tx/a1b2c3d4e5f6/"
  ],
  "scrapeDetails": true
}
```

### Example: fast search without detail pages

```json
{
  "mode": "search",
  "searchLocation": "Chicago, IL",
  "scrapeDetails": false,
  "maxItems": 200,
  "maxPages": 10
}
```

## Use cases

- Compare rent ranges and amenities across neighborhoods before relocating.
- Filter for pet-friendly or amenity-specific rentals in a target city.
- Collect leasing office contact details for outreach to property managers.
- Track per-floor-plan pricing for a specific property over time.
- Study walkability and transit scores alongside rent for housing research.

## FAQ

**Is this affiliated with Apartments.com?**
No. This actor reads Apartments.com's own public listing pages and has no relationship with Apartments.com.

**Do I need an account to use this?**
No. Every listing comes from pages that are publicly visible without logging in.

**What's the difference between `scrapeDetails: true` and `false`?**
With `scrapeDetails: true`, the actor visits each listing's own page for the full amenity list, walkability scores, per-unit floor plans (`units`), description, and leasing phone number. With it `false`, you only get what the search results page itself shows — faster, but without those fields.

**Can I scrape a specific listing instead of searching?**
Yes — set `mode` to `urls` and list the listing (or search-result) URLs in `startUrls`.

**Why do some listings have no `latitude`/`longitude` or `walkScore`?**
Those come from data a listing's own page publishes; when a property doesn't include them, the record omits the field rather than sending an empty value.

**Why does `hasRentSpecials` only ever appear as `true`?**
It's only added to a record when the listing is actually advertising a current move-in deal; listings without one simply don't have the field.

**How many listings can one run return?**
Up to `maxItems`, capped at 500, limited further by how many listings turn up across `maxPages` of search results.

**How current is the data?**
Every run searches Apartments.com live, so rent, availability, and specials reflect what's listed at run time.
