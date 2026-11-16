# SpareRoom UK Flatshare Budget & Radius Search

SpareRoom is the UK's largest site for rooms in shared houses, flats and studios. This actor runs a SpareRoom room search for a city with a budget range and a search radius from the centre, or fetches specific listing URLs directly, and turns every advert into a structured record: rent, bills-included status, neighbourhood, property type, landlord type, availability and photo.

## What this actor does

- Searches rooms to rent in any UK city or area, with min/max monthly budget and a 0–50 mile radius from the city centre
- Walks search result pages automatically until `maxItems` is reached or listings run out
- Fetches full detail for a specific list of SpareRoom listing URLs instead of running a search
- Surfaces landlord signals — advertiser role, profile photo, listing age and SpareRoom's verified-advertiser badge
- Empty fields are omitted from records — a field only appears when the advert actually shows that data

## Output per room listing

- `roomId` — SpareRoom listing ID
- `title` — advertised headline
- `price` — advertised rent as a number
- `priceType` — the period the price is quoted in (e.g. per month or per week)
- `city` — the city or area searched
- `area` — neighbourhood
- `postcode` — postcode or partial postcode, when published
- `propertyType` — e.g. flat, house
- `roomsInProperty` — total rooms in the property
- `landlordType` — advertiser role, e.g. live-in landlord, agent, current tenant
- `landlordPhotoUrl` — advertiser profile photo
- `listingType` — advert type as classified by SpareRoom
- `billsIncluded` — `true` when bills are advertised as included
- `availableNow` — `true` when the room is available immediately
- `availableFrom` — availability text as advertised
- `daysOld` — how long the advert has been live
- `isVerified` — SpareRoom's advertiser verification flag
- `description` — listing description (fetched in `roomDetails` mode)
- `furnishing` — furnished / unfurnished (fetched in `roomDetails` mode)
- `imageUrl` — main listing photo
- `roomUrl` — the listing's page on SpareRoom
- `scrapedAt` — UTC timestamp when the record was collected

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `searchRooms` | `searchRooms`, `searchByCity`, and `searchByBudget` all run the identical city/budget/radius search; `roomDetails` fetches specific URLs instead |
| `city` | string | `London` | City or area to search, e.g. `Manchester`, `Edinburgh` |
| `maxBudget` | integer | `1500` | Maximum monthly rent in GBP (100–10,000) |
| `minBudget` | integer | – | Minimum monthly rent in GBP |
| `milesFromCenter` | integer | `3` | Search radius from the city centre, in miles (0–50) |
| `roomType` | string | `any` | `any`, `double`, `single`, `ensuite`, or `studio` — accepted by the schema but not currently narrowed server-side; filter the output on `title`/`propertyType` if you need this |
| `roomUrls` | array | `[]` | SpareRoom listing URLs to fetch directly, for `roomDetails` mode |
| `maxItems` | integer | `20` | Maximum listings to return (1–500) |

### Example: London rooms under £1,200 within 5 miles

```json
{
  "mode": "searchRooms",
  "city": "London",
  "maxBudget": 1200,
  "milesFromCenter": 5,
  "maxItems": 100
}
```

### Example: a budget band in Manchester

```json
{
  "mode": "searchByBudget",
  "city": "Manchester",
  "minBudget": 500,
  "maxBudget": 800,
  "maxItems": 200
}
```

### Example: a wider radius around a smaller city

```json
{
  "mode": "searchByCity",
  "city": "Edinburgh",
  "maxBudget": 900,
  "milesFromCenter": 15
}
```

### Example: fetching specific listings

```json
{
  "mode": "roomDetails",
  "roomUrls": [
    "https://www.spareroom.co.uk/flatshare/flatshare_detail.pl?flatshare_id=15891234"
  ]
}
```

## Use cases

- **Flat-hunting tools** — surface rooms within budget and commute radius of a target address
- **Rental market tracking** — monitor asking rents by city and neighbourhood over successive runs
- **Buy-to-let research** — benchmark room rents in a target postcode before purchasing an investment property
- **Letting agency monitoring** — watch how quickly a competitor's adverts turn over using `daysOld`
- **Flatshare market analysis** — quantify how bills-included pricing and landlord type vary across a city

## FAQ

**Is this affiliated with SpareRoom?** No. This is an independent, third-party actor that reads SpareRoom's publicly visible search and listing pages; it is not operated by or affiliated with SpareRoom.co.uk.

**Do I need a SpareRoom account?** No — only publicly visible listing data is collected. Contact details hidden behind SpareRoom's own login are not returned.

**Is `price` quoted per month or per week?** Check the paired `priceType` field — advertisers quote both, and the actor reports the price exactly as shown rather than converting it. The `minBudget`/`maxBudget` inputs are still applied on a monthly basis by SpareRoom's own search.

**Does setting `roomType` actually filter the results?** Not yet — the field is accepted by the input schema, but search results aren't currently narrowed by it server-side. Filter the output records on `title` or `propertyType` if you need to isolate a room type.

**Why do some listings have no postcode?** Many advertisers only publish a neighbourhood name or a partial postcode to protect their privacy; when no postcode is shown, the field is left out rather than guessed.

**What does `landlordType` tell me?** Whether the advert comes from a live-in landlord, a current tenant looking for a flatmate, or a letting agent — useful as a proxy for how the tenancy will be handled.

**What does a high `daysOld` value usually mean?** It means the advert has been live a while without being taken down, which can indicate the room isn't renting easily — sometimes a sign there's room to negotiate on price.

**Can I search outside the UK?** No — SpareRoom's UK site is the data source, so coverage spans England, Scotland, Wales and Northern Ireland only.

**How fresh is the data?** Everything is fetched live when the actor runs, so prices and availability reflect SpareRoom at that moment.
