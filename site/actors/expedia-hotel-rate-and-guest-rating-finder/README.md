# Expedia Hotel Rate & Guest Rating Finder

Search Expedia hotels by destination or paste hotel and search links, and get live nightly and total rates, exact star class, guest ratings, review counts, address and coordinates, amenities, photos, and optional room-by-room rates with cancellation policies. Results come back as clean JSON or CSV — no API key, login, or proxy setup required.

## What this actor does

- **Two modes:** `search` for a destination, or `urls` for Expedia hotel pages, Hotel-Search links, share links, and bare hotel IDs.
- **Live rates:** nightly price, total stay price with taxes and fees, original strike-through price, and currency.
- **Quality signals:** guest rating and review count from the search card, plus exact star class (including half-stars) from the property page.
- **Property detail:** address, latitude/longitude, amenity highlights, policies, check-in/check-out times, description, photo gallery, and FAQs.
- **Optional room data:** room types, individual rate plans, payment timing, refundability, and cancellation policy text.
- **Filters:** sort order, star class, guest rating, nightly price, property type, amenities, free cancellation, availability, and property name.
- Every field is omitted when Expedia does not provide it — the dataset never contains `null`, empty strings, or sentinel rows.

## Output per hotel

- `type` — always `hotel_expedia`
- `hotelId` — Expedia property identifier
- `name` — hotel name
- `url` — Expedia hotel page
- `price` — legacy nightly price text (e.g. `$751 nightly`)
- `totalPrice` — legacy total price text (e.g. `$1,833 total`)
- `pricePerNight`, `priceTotal`, `priceStrikethrough` — numeric prices
- `priceCurrency` — currency code (USD on expedia.com)
- `rating` — legacy guest rating text (0–10)
- `guestRating` — numeric guest rating
- `ratingText` — label such as `Excellent` or `Wonderful`
- `reviewCount` — legacy review count text
- `guestReviewCount` — numeric review count
- `starRating` — exact star class, half-stars possible
- `neighborhood` — area shown on the hotel card
- `address` — `full`, `street`, `city`, `region`, `postalCode`, `country`
- `latitude`, `longitude` — property coordinates
- `amenities` — amenity highlights from the property page
- `topAmenities` — amenities highlighted on the search card
- `imageUrl`, `images` — main photo and gallery
- `badges` — labels such as `VIP Access`
- `freeCancellation` — whether a refundable rate exists
- `cancellationPolicy` — policy text for the cheapest or first rate
- `roomRates` — room types and rate plans, each with `roomName`, `ratePlanId`, `pricePerNight`, `priceTotal`, `priceCurrency`, `paymentPolicy`, `cancellationPolicy`, `refundable`, `available`, `roomsLeft`
- `checkInTime`, `checkOutTime` — property policy times
- `propertyDescription` — overview or editorial description
- `propertyType` — property type when shown
- `policies`, `importantInfo` — policy and fee text
- `faqs` — property questions and answers
- `description` — short card tagline when Expedia shows one
- `destination`, `resolvedDestination`, `regionId` — search destination and Expedia resolution
- `checkIn`, `checkOut`, `adults`, `rooms`, `childrenAges` — search parameters used
- `searchUrl`, `searchPosition`, `isSponsored`, `scrapedAt`

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `search` | `search` for a destination, or `urls` for links and IDs. |
| `destination` | string | – | City, area, landmark, airport, or hotel name (mode=search). Prefilled with `New York` for the daily example. |
| `destinationId` | string | – | Optional Expedia region ID to skip destination lookup (mode=search). |
| `startUrls` | array | `[]` | Hotel pages, Hotel-Search links, share links, or bare IDs such as `h17117062` (mode=urls). |
| `checkIn` | string | – | `YYYY-MM-DD`; empty uses 30 days from today. |
| `checkOut` | string | – | `YYYY-MM-DD`; empty uses 2 nights after check-in. |
| `adults` | integer | `2` | Adults per room, 1–14. |
| `childrenAges` | array | `[]` | One age (0–17) per child; numbers or quoted strings are accepted. |
| `rooms` | integer | `1` | Number of rooms, 1–8. |
| `maxItems` | integer | `20` | Hotel record cap, 1–1000. Expedia returns up to 500 per request; larger caps use pagination. |
| `sortBy` | string | `recommended` | `recommended`, `priceLowToHigh`, `priceHighToLow`, `distance`, `guestRating`, or `starRating`. |
| `starRatings` | array | `[]` | Star classes 1–5; empty includes all. |
| `minGuestRating` | string | `any` | `any`, `7+`, `8+`, or `9+`. |
| `minPrice` | number | – | Minimum nightly price in USD. |
| `maxPrice` | number | – | Maximum nightly price in USD. |
| `propertyType` | string | `any` | `any`, `hotels`, `vacationRentals`, `hotel`, `motel`, `resort`, `apartment`, `hostel`, `bedAndBreakfast`, `guesthouse`, `villa`, `vacationHome`, `lodge`, `cottage`, or `cabin`. |
| `amenities` | array | `[]` | Only hotels with all selected amenities. |
| `freeCancellationOnly` | boolean | `false` | Only fully refundable properties. |
| `availableOnly` | boolean | `false` | Only properties with availability for the dates. |
| `propertyName` | string | – | Filter by hotel name, for example Marriott. |
| `includePropertyDetails` | boolean | `true` | Add exact stars, address, coordinates, amenity highlights, policies, and gallery. |
| `includeRoomRates` | boolean | `false` | Add room types, rate plans, payment, and cancellation policies. |

### Example: weekend rates in New York

```json
{
  "mode": "search",
  "destination": "New York",
  "checkIn": "2026-11-10",
  "checkOut": "2026-11-12",
  "adults": 2,
  "maxItems": 20,
  "starRatings": ["4", "5"],
  "sortBy": "priceLowToHigh",
  "includePropertyDetails": true
}
```

### Example: family stay with two rooms

```json
{
  "mode": "search",
  "destination": "Orlando, Florida",
  "checkIn": "2026-12-20",
  "checkOut": "2026-12-27",
  "adults": 2,
  "childrenAges": ["5", "9"],
  "rooms": 2,
  "maxItems": 30,
  "freeCancellationOnly": true,
  "includeRoomRates": true
}
```

### Example: exact hotel with room rates

```json
{
  "mode": "urls",
  "startUrls": [
    "https://www.expedia.com/New-York-Hotels-PUBLIC.h17117062.Hotel-Information",
    "h19681371"
  ],
  "checkIn": "2026-11-10",
  "checkOut": "2026-11-12",
  "includeRoomRates": true
}
```

### Example: reuse an Expedia search link

```json
{
  "mode": "urls",
  "startUrls": [
    "https://www.expedia.com/Hotel-Search?destination=London&startDate=2026-11-20&endDate=2026-11-22&adults=2&rooms=1&star=40"
  ],
  "maxItems": 50,
  "includePropertyDetails": true
}
```

## Use cases

- **Rate shopping** — compare nightly and total prices for the same dates and occupancy.
- **Revenue and pricing analysis** — track star class, guest ratings, and rate spreads across a market.
- **Travel planning** — shortlist hotels by refundability, amenities, and location.
- **Market research** — snapshot a city's hotel inventory, ratings, and price distribution.
- **Property monitoring** — rerun the same hotel or search link to watch rates and policies over time.
- **Data enrichment** — join hotel IDs, coordinates, ratings, and addresses to an existing travel dataset.

## FAQ

**Does this actor need an API key, login, or proxy?**
No. It reads public Expedia hotel pages and selects a working connection automatically. Just fill in the input and run it.

**How many hotels can one run return?**
Expedia returns up to 500 hotels per search request. The actor follows the result cursor to fetch deeper pages, up to the 1,000-hotel `maxItems` cap. A destination that genuinely has fewer properties returns fewer rows.

**Why do some records have no star rating or address?**
Those fields come from the individual property page. They are present when `includePropertyDetails` is on and Expedia returns the page. If a property page is temporarily unavailable, the record still carries the search-card fields and the missing values are omitted rather than fabricated.

**Why do I need to enable room rates separately?**
Room types and rate plans require one extra Expedia read per hotel. Leave `includeRoomRates` off for a fast, cheap search; turn it on when you need payment timing and cancellation policy text.

**Are prices per night or for the whole stay?**
Both are returned. `pricePerNight` is the nightly amount; `priceTotal` is the stay total including taxes and fees. The legacy `price` and `totalPrice` strings mirror what Expedia displayed.

**What does the guest rating scale mean?**
`guestRating` is Expedia's own score out of 10. `ratingText` is the label Expedia shows, such as Wonderful or Excellent.

**Can I get review text with this actor?**
No. This actor returns the guest rating and review count shown on the property, not individual review text. That keeps runs fast and focused on rates and ratings.

**Are all Expedia regional sites supported?**
No. This actor reads expedia.com and returns USD prices. Regional Expedia domains and other currencies are not part of this version.

**What happens if Expedia blocks the run?**
The actor retries with fresh sessions and fails with a clear status instead of returning an empty dataset or placeholder rows.

**Is this affiliated with Expedia?**
No. This is an independent third-party actor that reads publicly visible Expedia hotel data.

## Limitations

- Review text, regional Expedia sites/currencies, and hotel brand colors are not part of this actor.
- App share links are accepted when they resolve to a hotel, but public app links often expire; hotel pages, search links, and IDs are the reliable inputs.
- `maxItems` is capped at 1,000. Expedia returns 500 per request and may show fewer real hotels than its matched count because of ads and placeholders.
