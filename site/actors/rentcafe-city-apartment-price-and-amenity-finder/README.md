# RentCafe City Apartment Price & Amenity Finder

RentCafe.com lists apartment communities across the US with per-unit-type pricing, full amenity lists, star ratings, phone numbers and management-company contacts. This actor searches a city — or a set of saved RentCafe search URLs — and returns each community as a structured record for rent comparison, market research or a property-management lead list.

## What this actor does

- Searches any US city RentCafe recognises by its URL slug, or accepts full RentCafe city search URLs
- Uses RentCafe's own server-side filters: bedrooms, bathrooms, rent range, minimum rating, pet policy, amenity, property type, rent specials and move-in date
- Orders results with RentCafe's own sort: best match, price low/high, rating high, newest, or 3D-tour listings
- Paginates until your `maxItems` is reached or RentCafe's public result list ends
- Merges the visible result card with RentCafe's schema.org JSON-LD so each record carries both per-unit prices and the full amenity/contact data
- Omits empty fields — a field appears only when RentCafe published real data for that community

## Output per community

- `propertyId`, `name`, `url` — RentCafe's community ID, name and listing URL
- `street`, `city`, `state`, `zip`, `fullAddress`
- `latitude`, `longitude`
- `telephone` — leasing phone number from the listing's structured data
- `rating`, `reviewCount` — star rating and number of reviews
- `bedPricePairs` — array of `{beds, startingPrice}` for every unit type advertised
- `minPrice`, `maxPrice` — lowest and highest starting price across unit types
- `minBeds`, `maxBeds` — smallest and largest advertised bedroom count
- `amenities` — full amenity list from the listing's structured data
- `amenitySummary` — the shorter amenity line shown on the result card
- `priceCurrency`, `availability`
- `badges` — promo tags such as "Rent specials" or "New Units Available"
- `verifiedListing` — `true` when RentCafe marks the listing as verified
- `photoUrl` — full-size listing photo
- `managementCompany` — property management company, when shown
- `locationSlug` — the normalized city slug used for the search
- `filterApplied` — the path filter segment used for this run, if any
- `scrapedAt` — UTC timestamp when the record was collected

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | select | `city` | `City search` builds a search from `location`; `Search URLs` scrapes the RentCafe search URLs in `searchUrls` |
| `location` | string | – | City as it appears on RentCafe, e.g. `austin-tx`, `Austin, TX`, or `new-york-city-ny`; a full RentCafe city search URL also works. Required in city mode |
| `searchUrls` | array | – | RentCafe city search URLs, e.g. `https://www.rentcafe.com/apartments-for-rent/austin-tx/pet-friendly/`. Required in URL mode |
| `beds` | select | Any | Studio, 1, 2, 3 or 4+ bedrooms |
| `bathrooms` | select | Any | 1, 2 or 3+ bathrooms |
| `petPolicy` | select | Any | Pet friendly, cats, dogs, or no pets |
| `amenity` | select | Any | One amenity from RentCafe's 21-option amenity filter (pool, garage, washer/dryer, …) |
| `propertyType` | select | Any | Apartment, condo, home, townhouse, student or affordable |
| `minPrice` / `maxPrice` | integer | – | Monthly rent range, applied by RentCafe and re-checked on each record |
| `minRating` | integer | – | Drop communities rated below this (0–5). 3–5 is also sent to RentCafe's rating filter |
| `rentSpecialsOnly` | boolean | `false` | Only communities currently advertising a rent special |
| `moveInDate` | string | – | Optional `YYYY-MM-DD` move-in date |
| `sort` | select | Best match | Best match, price low/high, rating high, newest or 3D tour |
| `maxItems` | integer | `25` | Maximum communities to return (1–2000) |

### Example: 2-bedroom, pet-friendly Austin communities with a pool, rated 4+, $1,000–$2,500

```json
{
  "mode": "city",
  "location": "austin-tx",
  "beds": "2",
  "petPolicy": "pet-friendly",
  "amenity": "c24",
  "minPrice": 1000,
  "maxPrice": 2500,
  "minRating": 4,
  "sort": "price-low",
  "maxItems": 40
}
```

### Example: scrape two saved RentCafe searches

```json
{
  "mode": "urls",
  "searchUrls": [
    "https://www.rentcafe.com/apartments-for-rent/austin-tx/pet-friendly/?Beds=Two",
    "https://www.rentcafe.com/apartments-for-rent/austin-tx/under-1000/"
  ],
  "maxItems": 100
}
```

## Use cases

- **Rent comparison** — pull every community in a city for a bedroom count and compare starting prices side by side
- **Lease shopping** — narrow by rent range, pet policy, pool, garage or fitness center before you start calling
- **Property-management leads** — collect community names, phone numbers and management companies across a city
- **Market research** — track community-level `minPrice`/`maxPrice` and amenity coverage over time
- **Saved-search automation** — paste an existing RentCafe search URL and turn it into a daily dataset

## FAQ

**How do I find the city slug?** The path segment right after `/apartments-for-rent/` on RentCafe.com is the slug (e.g. `rentcafe.com/apartments-for-rent/austin-tx/` → `austin-tx`). You can also type `City, ST` or paste the full search URL.

**Can I combine filters?** Yes. The new filters use RentCafe's own server-side parameters and combine with each other. The old single path-segment filters still work for saved tasks, but the filter fields in the input form are the supported way to stack several conditions.

**Do I need a login or proxy?** No. The listing pages are public and the actor needs no account, API key or proxy setup. It picks the network route it needs automatically.

**Why are there fewer records than the "Rentals Available" number?** RentCafe's public city search paginates *communities* (usually 25 per page), while its rental count includes individual units/floor plans. The actor returns communities and tells you in the status message when the public result list is exhausted.

**Why does a listing URL return 403 to curl?** RentCafe blocks some non-browser requests to property pages. The URLs are valid and open normally in a browser; photos and the dataset fields are unaffected.

**Why do some communities have no price or rating?** RentCafe only shows what each property manager supplied. A community can be listed with "contact for pricing" and no reviews yet; those fields are omitted rather than filled with a guess.

**Is this affiliated with RentCafe?** No. This is an independent actor that reads RentCafe's publicly available listing pages; it is not operated by or affiliated with RentCafe/Yardi.
