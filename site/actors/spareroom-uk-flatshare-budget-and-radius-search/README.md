# SpareRoom UK Flatshare Budget & Radius Search

SpareRoom is the UK's busiest flatshare site. This actor turns its public room adverts into structured data. Run a location search with a monthly budget, radius and optional filters, or paste your own listing URLs to collect the full advert detail page.

Everything is collected from public, logged-out pages. No SpareRoom account, cookie or proxy setup is required.

## Modes

| Mode | What it does |
|---|---|
| `search` | Resolves a UK city, town, neighbourhood or postcode on SpareRoom, applies your filters and pages through the results. |
| `details` | Fetches one or more SpareRoom listing URLs and returns the full advert detail record. |

The old mode names `searchRooms`, `searchByCity`, `searchByBudget` and `roomDetails` are still accepted and mapped to the matching mode, so existing saved tasks keep working.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | select | `search` | `search` or `details`; legacy mode names are accepted |
| `city` | text | `London` | UK city, area, postcode or `/flatshare/...` URL |
| `maxItems` | integer | `20` | Maximum listings to return (1–1000) |
| `maxBudget` | integer | `1500` | Maximum monthly rent in GBP |
| `minBudget` | integer | – | Minimum monthly rent in GBP |
| `milesFromCenter` | integer | `3` | Search radius in miles |
| `roomType` | select | `any` | any, double, single, ensuite or studio |
| `propertyType` | select | `any` | any, roomsInShares, studios or wholeProperties |
| `billsIncluded` | boolean | `false` | Bills included only |
| `furnished` | select | `any` | any, furnished or unfurnished |
| `postedBy` | select | `any` | any, privateLandlords, flatmates, liveIn or noLiveIn |
| `sortBy` | select | `relevance` | relevance, newest, lastUpdated, priceLow or priceHigh |
| `keyword` | text | – | Keyword filter |
| `moveInDate` | text | – | Move-in date in YYYY-MM-DD format |
| `shortLet` | boolean | `false` | Short lets only |
| `photosOnly` | boolean | `false` | Photo adverts only |
| `freeToContact` | boolean | `false` | Free-to-contact adverts only |
| `noDeposit` | boolean | `false` | No/low-deposit adverts only |
| `petsAllowed` | boolean | `false` | Pets considered only |
| `parking` | boolean | `false` | Parking only |
| `disabledAccess` | boolean | `false` | Disabled access only |
| `sharedLivingRoom` | boolean | `false` | Shared living room only |
| `minBeds` | select | – | Minimum bedrooms in the property |
| `maxBeds` | select | – | Maximum bedrooms in the property |
| `roomUrls` | stringList | `[]` | SpareRoom listing URLs for `details` mode |


## What you can filter on

- Monthly rent range (`minBudget` / `maxBudget`)
- Search radius in miles (`milesFromCenter`)
- Room type: double, single, en-suite or studio/1-bed
- Property type: rooms in existing shares, studio/1-bed flats or whole properties
- Bills included, furnished/unfurnished, posted-by role
- Keyword, move-in date, short lets
- Photo adverts only, free to contact, no/low deposit
- Pets considered, parking, disabled access, shared living room
- Bedroom-count range
- Sort order: newest, recently updated, price low/high

SpareRoom shows promoted adverts at the top of some searches. The actor removes promoted rows that visibly contradict the requested budget, room type, property type, poster role, bills, photo, free-to-contact or bedroom filters, then keeps paging to fill `maxItems`.

## Output per room

Every record uses the same field names in both modes; fields the advert does not publish are omitted rather than filled with placeholders.

| Field | Meaning |
|---|---|
| `roomId` | SpareRoom advert reference number |
| `title` | Advert headline |
| `price` | Rent exactly as advertised |
| `priceType` | `pcm` (per calendar month) or `pw` (per week) |
| `priceNormalised` | SpareRoom's normalised price when the page provides one |
| `priceNormalisedType` | Period for `priceNormalised` |
| `city` | Location you searched (search mode) |
| `area` | Neighbourhood or area shown in the advert |
| `postcode` | Full or partial postcode |
| `propertyType` | Property type as shown by SpareRoom |
| `propertyTypeDetail` | Extra property-type text, for example `studio flat` |
| `roomType` | Room type shown on the card, for example `Double room` |
| `roomsInProperty` | Total bedrooms in the property |
| `housemates` | Current number of flatmates/housemates (detail pages) |
| `landlordType` | Advertiser role, for example `live in landlord`, `agent`, `current flatmate` |
| `listingType` | Advert type, usually `offered` |
| `listingStatus` | Status badges such as `new today`, `boosted`, `now let` |
| `billsIncluded` | `true` / `false` when the advert states it |
| `deposit` / `depositCurrency` | Deposit amount and currency (detail pages) |
| `availableNow` | `true` when the advert says available now |
| `availableFrom` | Availability text exactly as advertised |
| `minimumTerm` / `maximumTerm` | Minimum and maximum stay (detail pages) |
| `shortLet` | `true` when short lets are considered |
| `furnishing` | Furnished / unfurnished (detail pages) |
| `livingRoom` | Living-room arrangement (detail pages) |
| `parking`, `garage`, `garden`, `balcony`, `disabledAccess`, `wifi` | Amenity flags (detail pages) |
| `householdAges`, `householdSmoker`, `householdPets`, `householdOccupation`, `householdGender` | Current household (detail pages) |
| `couplesOk`, `smokingOk`, `petsSuitable`, `referencesRequired`, `minAge`, `maxAge` | New-flatmate preferences (detail pages) |
| `description` | Advert text (full text in details mode, snippet in search mode) |
| `imageUrl` / `imageUrls` | Main photo and full detail-page photo gallery |
| `landlordPhotoUrl` | Advertiser profile photo, when published |
| `advertiserName`, `advertiserRole`, `advertiserUpgraded`, `advertiserVerified`, `advertiserLastActive`, `advertiserResponseRate` | Advertiser information |
| `isVerified`, `isFeatured`, `isFreeToContact`, `hasVideo`, `photoCount`, `daysOld` | Listing signals from the search card |
| `roomUrl` | Canonical advert URL |
| `scrapedAt` | UTC collection timestamp |

## Examples

### London rooms up to £1,200 within 5 miles

```json
{
  "mode": "search",
  "city": "London",
  "maxBudget": 1200,
  "milesFromCenter": 5,
  "maxItems": 100
}
```

### Double rooms with bills included in Manchester

```json
{
  "mode": "search",
  "city": "Manchester",
  "minBudget": 500,
  "maxBudget": 900,
  "roomType": "double",
  "billsIncluded": true,
  "sortBy": "priceLow",
  "maxItems": 50
}
```

### Fetch specific adverts

```json
{
  "mode": "details",
  "roomUrls": [
    "https://www.spareroom.co.uk/flatshare/london/chiswick/17509965",
    "https://www.spareroom.co.uk/flatshare/flatshare_detail.pl?flatshare_id=17454753"
  ]
}
```

### Legacy task shape (still supported)

```json
{
  "mode": "searchByBudget",
  "city": "Birmingham",
  "maxBudget": 700,
  "roomType": "any",
  "milesFromCenter": 3,
  "maxItems": 20
}
```

## Pagination and source limits

Search results are read 10 adverts at a time until `maxItems` is reached or the advert list ends. SpareRoom itself caps a single search at **1,000 results** (it displays `Showing first 1000 of ... ads`). When that happens the actor tells you in the run status message. To collect more, split the search by budget band, radius, room type or property type.

## FAQ

**Do I need a SpareRoom login?** No. The actor reads the same public search and advert pages a logged-out visitor sees.

**Can I get the advertiser's email or phone number?** No. Contact details are behind SpareRoom's login and are not collected.

**Why is `priceType` sometimes `pw`?** Some advertisers quote weekly rent. The actor reports the price exactly as advertised; use `priceNormalised` when SpareRoom provides a monthly equivalent.

**Why do some records have fewer fields?** Detail-page fields such as deposit, amenities and household information only exist when the advertiser published them. Missing fields are omitted, never guessed.

**Why do search results sometimes include a promoted advert outside my radius?** SpareRoom inserts promoted adverts at the top of some result pages. The actor removes promoted rows that clearly break the selected budget, room type, property type, poster, bills, photo, free-to-contact or bedroom filters, but it cannot calculate distance from a promoted advert's postcode, so a promoted row can still sit outside the radius.

**Is this affiliated with SpareRoom?** No. This is an independent third-party actor that reads publicly visible pages from SpareRoom.co.uk.

## Data source

Public SpareRoom.co.uk pages:

- `/flatshare/<location>/` search results with SpareRoom's own filter parameters
- `/flatshare/search.pl` location resolution
- `/flatshare/<area>/<id>` and `/flatshare/flatshare_detail.pl?flatshare_id=<id>` advert pages

## Limitations

- UK only. US listings, rooms-wanted adverts and buddy-up adverts are not part of this actor.
- SpareRoom caps each search at 1,000 results; refine filters to reach more.
- Contact details, email alerts and monitoring for new listings are not included.
- A promoted advert can appear outside the chosen radius; the actor filters the other fields it can verify from the card.
