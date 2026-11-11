# Trip.com Hotel Search & Nightly Price Finder

Search Trip.com for hotels in a city and date range, or pull the full listing for one specific hotel by its Trip.com ID. Each result carries the nightly price, star class, guest score, amenities, and a direct booking link so you can compare stays without opening the site.

## What this actor does

- **Two modes:** `searchHotels` for a city + date search, `hotelDetail` for a single hotel by its Trip.com hotel ID
- **Date-aware pricing:** pass check-in/check-out dates and get the price quoted for that exact stay; leave them blank and the actor defaults to tomorrow night
- **Quality filters:** minimum star rating, minimum guest score, and a maximum nightly price, applied before `maxItems` is counted
- **Location and amenities:** address, neighborhood, coordinates, and a cleaned list of hotel facilities when Trip.com publishes them
- **Empty fields are omitted** from each record — a hotel with no listed amenities or coordinates simply won't carry those keys

## Output per hotel

- `hotelId` — Trip.com's internal hotel ID
- `name` — hotel name
- `address` — full street address
- `neighborhood` — district/area name, when Trip.com lists one
- `starRating` — official star class (1–5)
- `rating` — guest score on Trip.com's 0–10 scale
- `reviewCount` — number of guest reviews behind the score
- `pricePerNight` — quoted nightly price, excluding taxes/fees
- `priceWithTax` — nightly price including taxes and fees, when Trip.com supplies it separately
- `currency` — currency code for the price fields
- `priceExplanation` — Trip.com's price breakdown label (e.g. "Total incl. taxes & fees: $277")
- `paymentPolicy` — payment terms text (e.g. pay-at-hotel vs. pay-now), when published
- `amenities[]` — hotel facility names (Wi-Fi, pool, gym, etc.)
- `url` — Trip.com hotel detail/booking page
- `thumbnailUrl` — hotel photo URL
- `lat`, `lon` — geographic coordinates
- `checkIn`, `checkOut` — the dates the search used (mode=`searchHotels` only)
- `city` — the searched city name (mode=`searchHotels` only)
- `recordType` — always `"hotel"`
- `scrapedAt` — ISO 8601 timestamp of when the record was collected

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `searchHotels` | `searchHotels` or `hotelDetail` |
| `hotelId` | string | – | Trip.com hotel ID to fetch (mode=`hotelDetail`) |
| `cityName` | string | `London` | City to search; best coverage is Europe, Asia, and the Middle East (mode=`searchHotels`) |
| `checkIn` | string | – | Check-in date, `YYYY-MM-DD`; defaults to tomorrow if blank |
| `checkOut` | string | – | Check-out date, `YYYY-MM-DD`; defaults to the night after check-in if blank |
| `minStars` | integer | – | Minimum star rating, 1–5 |
| `minRating` | number | – | Minimum guest score, 0–10 |
| `maxPrice` | number | – | Maximum nightly price in USD |
| `maxItems` | integer | `20` | Maximum hotels to return (1–200) |
| `proxyConfiguration` | object | `{"useApifyProxy": true}` | Apify Proxy settings used to fetch Trip.com's search results |

### Example: search London with a quality and price filter

```json
{
    "mode": "searchHotels",
    "cityName": "London",
    "checkIn": "2026-06-10",
    "checkOut": "2026-06-13",
    "minStars": 4,
    "minRating": 8.0,
    "maxPrice": 300,
    "maxItems": 30
}
```

### Example: broad search for a destination, no filters

```json
{
    "mode": "searchHotels",
    "cityName": "Bangkok",
    "maxItems": 50
}
```

### Example: fetch one hotel by its Trip.com ID

```json
{
    "mode": "hotelDetail",
    "hotelId": "732036"
}
```

## Use cases

- **Price tracking** — rerun the same city and date range on a schedule to watch nightly rates move
- **Shortlist building** — filter by `minStars` and `minRating` to narrow a destination down to hotels worth considering
- **Rate comparison** — pull `hotelDetail` for a known shortlist of properties and compare their current quoted price
- **Travel content research** — gather amenities, star class, and guest scores for a city to build comparison tables or articles
- **Budget planning** — cap results with `maxPrice` to see what's available within a travel budget

## FAQ

**Do I need a Trip.com account?**
No. Hotel search and detail pages are publicly accessible without logging in.

**What currency are prices in?**
`currency` reflects whatever Trip.com returns for that listing; when Trip.com doesn't specify one, the actor reports `USD`.

**What's the difference between `pricePerNight` and `priceWithTax`?**
`pricePerNight` is the base nightly rate; `priceWithTax` is the same stay including taxes and fees, when Trip.com exposes that figure separately. If Trip.com only shows one number, only `pricePerNight` is populated.

**What happens if I leave check-in/check-out blank?**
The actor searches for tomorrow night by default, and check-out defaults to the night after check-in if only one date is given.

**Why do some cities in the US return no hotels?**
Trip.com serves North American markets through a different web frontend that doesn't expose hotel listings in the format this actor reads. European, Asian, and Middle Eastern cities are reliably supported.

**Why is `hotelId` needed for the detail mode instead of a URL?**
Trip.com's hotel detail pages are addressed by numeric ID; you can find it in any hotel's Trip.com URL (`?hotelId=...`) or from a prior `searchHotels` run's output.

**Why might `amenities` or `neighborhood` be missing for a hotel?**
Trip.com doesn't publish the same level of detail for every property — when a field isn't available on the listing, it's left out of the record rather than returned empty.

**Is this affiliated with Trip.com?**
No. This is an independent, third-party actor and is not operated by or affiliated with Trip.com / Ctrip.
