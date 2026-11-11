# VivaReal Brazil Real Estate Listings by City & Price

Scrape **VivaReal Brazil** (vivareal.com.br) for homes for sale or rent in any Brazilian city. Records carry the BRL price, condo fee, IPTU, usable and total area, room counts, street address and coordinates, amenities, every listing photo, the advertiser's public contact details, and a working listing URL whenever VivaReal publishes them. Search by city, state, and neighborhood, or paste specific listing URLs/IDs to get full titles, descriptions, and detail-fresh fields. Search records come from VivaReal's search index; `byUrls` reads the detail endpoint and is the freshest source for descriptions, images, contact details, and timestamps.

> VivaReal Colombia (vivareal.com.co) has been offline since 2024 and returns HTTP 410. The live VivaReal marketplace is the Brazilian one. This actor's historical slug still says "Colombia", but it now scrapes VivaReal **Brazil** only. Bogotá or other Colombian inputs return zero records with a clear status message — no invented data.

## What this actor does

- **Three modes:** `sale`, `rent`, and `byUrls` (VivaReal listing URLs or numeric IDs).
- **City searches:** combine city + state, optionally narrow to a neighborhood.
- **Server + client filters:** price, usable area, bedrooms, bathrooms, parking, property type, usage type, listing type (`USED`, `DEVELOPMENT`, or both), and amenities.
- **Complete property record:** condo fee, IPTU with period, usable and total area, suites, coordinates, amenities, all photos, advertiser name/licence/phones, WhatsApp, and timestamps (`createdAt` in search; `createdAt` + `updatedAt` in `byUrls`).
- **Deep pagination:** up to 1,000 listings per run, deduplicated by listing ID.
- **Empty fields are omitted** — a missing value is never emitted as `null`, `""`, or `[]`.

## Output per listing

| Field | Description |
|---|---|
| `listingId` | VivaReal listing identifier |
| `title` | Portal title when available (`byUrls`, or the search `h2Tag`); otherwise a composed title |
| `titleSource` | `portal` when the title came from VivaReal, `generated` when composed |
| `description` | Full description (mode=`byUrls` only; search results do not include it) |
| `price`, `currency` | Asking price and `BRL` |
| `condoFee` | Monthly condo fee in BRL, when published |
| `iptu`, `iptuPeriod` | IPTU value and period (`yearly` or `monthly`) |
| `area` | Usable area in m² |
| `totalArea` | Total area in m², when published |
| `bedrooms`, `bathrooms`, `suites`, `parkingSpaces` | Room and parking counts |
| `propertyType`, `propertyTypeLabel` | VivaReal code and Portuguese label (`APARTMENT` / `Apartamento`, etc.) |
| `usageType` | `RESIDENTIAL` or `COMMERCIAL` |
| `transactionType` | `sale` or `rental` from the listing's pricing block |
| `listingType` | `used` or `development` |
| `street`, `streetNumber`, `neighborhood`, `city`, `state`, `stateCode`, `zipCode` | Address parts |
| `address` | Readable address string built from the available parts |
| `latitude`, `longitude` | Exact map coordinates when available; approximate-location listings may omit them |
| `amenities` | Amenity codes such as `POOL`, `GYM`, `ELEVATOR`, `BARBECUE_GRILL` |
| `images` | Every listing photo, in source order |
| `imageUrl` | First photo |
| `advertiserName`, `advertiserLicense` | Advertiser/agency name and licence number |
| `advertiserPhones` | Public phone numbers |
| `whatsapp` | Public WhatsApp number, when published |
| `createdAt`, `updatedAt` | Listing timestamps; `updatedAt` is returned in `byUrls` and omitted in search (the search API does not return it) |
| `listingUrl` | `https://www.vivareal.com.br/imovel/<id>/` |
| `mode` | `sale`, `rent`, or `byUrls` |
| `recordType` | Always `property` |
| `scrapedAt` | ISO 8601 timestamp of the scrape |

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `sale` | `sale`, `rent`, or `byUrls` |
| `city` | string | `São Paulo` | Brazilian city to search |
| `state` | string | `SP` | Brazilian state (UF) |
| `neighborhood` | string | – | Optional neighborhood within the city |
| `propertyType` | string | `ANY` | Any type, apartment, house, penthouse, flat, kitnet, office, land, commercial, warehouse, garage, farm, or hotel |
| `usageType` | string | `ANY` | `ANY`, `RESIDENTIAL`, or `COMMERCIAL` |
| `listingType` | string | `USED` | `USED`, `DEVELOPMENT`, or `ALL` |
| `priceMin`, `priceMax` | integer | – | Price range in BRL |
| `bedroomsMin`, `bedroomsMax` | integer | – | Bedroom range |
| `bathroomsMin`, `bathroomsMax` | integer | – | Bathroom range |
| `parkingMin` | integer | – | Minimum parking spaces (search currently applies this as an exact match; see Limitations) |
| `areaMin`, `areaMax` | integer | – | Usable area range in m² |
| `amenities` | array | – | Required amenity codes (all must match) |
| `listingUrls` | array | – | VivaReal URLs or numeric IDs (mode=`byUrls`) |
| `maxItems` | integer | `50` | Maximum records to return (1–1000) |

Legacy saved tasks still work: `mode` values `searchForSale` / `searchForRent` are treated as `sale` / `rent`, and `propertyType` values `TERRAIN` / `COMMERCIAL` are treated as land / commercial property.

### Example: apartments for sale in São Paulo

```json
{
  "mode": "sale",
  "city": "São Paulo",
  "state": "SP",
  "propertyType": "APARTMENT",
  "priceMin": 400000,
  "priceMax": 900000,
  "bedroomsMin": 2,
  "areaMin": 60,
  "maxItems": 100
}
```

### Example: houses for rent in Rio de Janeiro

```json
{
  "mode": "rent",
  "city": "Rio de Janeiro",
  "state": "RJ",
  "neighborhood": "Copacabana",
  "propertyType": "HOME",
  "parkingMin": 1,
  "maxItems": 50
}
```

### Example: listing lookup by URL

```json
{
  "mode": "byUrls",
  "listingUrls": [
    "https://www.vivareal.com.br/imovel/2918839249/",
    "2918720977"
  ]
}
```

### Example: amenities and area in Belo Horizonte

```json
{
  "mode": "sale",
  "city": "Belo Horizonte",
  "state": "MG",
  "amenities": ["POOL", "GYM", "ELEVATOR"],
  "areaMin": 80,
  "areaMax": 200,
  "listingType": "USED",
  "maxItems": 30
}
```

## Use cases

- **Market research** — compare asking prices, condo fees, and IPTU across Brazilian cities and neighborhoods.
- **Lead generation** — build a current list of active listings with advertiser phone and WhatsApp contacts.
- **Investment screening** — filter by price, area, rooms, and amenities to shortlist candidate properties.
- **Rental monitoring** — track rent asking prices and availability in a target neighborhood.
- **Proptech data feeds** — power a search, valuation, or comparison product with live VivaReal inventory.

## FAQ

**Which cities can I search?**
Any Brazilian city covered by VivaReal, including São Paulo, Rio de Janeiro, Belo Horizonte, Curitiba, Porto Alegre, Salvador, Recife, Fortaleza, and Brasília. Pass the city name in `city` and its state in `state`.

**What currency are prices in?**
Brazilian reais (BRL). VivaReal Brazil lists prices in BRL. The actor does not convert values or apply legacy COP amounts.

**Why does the title look generated in search results?**
VivaReal's search API does not return a title for most listings, and some of the `h2Tag` titles it does return are truncated. The actor uses the portal title when one exists, otherwise it composes a readable title from the property type, bedroom count, neighborhood, and city, and marks it with `titleSource: "generated"`. Use `byUrls` for the full portal title.

**Why is `description` missing from search results?**
Descriptions are not part of VivaReal's search response. Use `mode: "byUrls"` with listing URLs or IDs to fetch full titles, descriptions, and updated timestamps.

**What does `iptuPeriod` mean?**
It tells you whether the IPTU value is yearly or monthly. Search mode follows the search API's `yearlyIptu` field; the detail response (and `byUrls`) can mark the same value monthly. The actor preserves the period from the response it used instead of silently converting it, so use `byUrls` for detail-consistent periods.

**How many listings can I get in one run?**
Up to 1,000 via `maxItems`. The actor pages through VivaReal at 30 listings per page. A very broad, rarity-filtered search can hit the 5,000-row scan limit; narrow the city, neighborhood, price, or property type for deeper coverage.

**Why are some fields missing from individual records?**
VivaReal does not publish every field for every listing, and approximate-location listings can lack a street number or exact coordinates. The actor omits a missing field rather than guessing or filling it with a placeholder. Search records can also lag the live detail page; use `byUrls` for the freshest values.

**Can I search VivaReal Colombia or Bogotá?**
No. VivaReal Colombia is offline. Colombian city names return zero records with a status message explaining that the actor now covers Brazil.

**Do I need a VivaReal account, login, or proxy?**
No. The actor reads public VivaReal listings and chooses a working connection automatically. Just fill in the input and run it.

**Is this affiliated with VivaReal?**
No. This is an independent, third-party actor that reads VivaReal's public listing data and is not operated by or affiliated with VivaReal.

## Limitations

- **No descriptions in search:** only `byUrls` returns the full description.
- **Composed titles:** most search records use a generated title because the search API omits the portal title.
- **Search-index freshness:** `sale`/`rent` records come from VivaReal's search index. Area, room counts, neighborhood, and IPTU period can differ from the live listing page; `byUrls` returns detail-fresh values. In the 2026-10-24 25-listing check, 2/15 sale records had stale area/room/parking values, 6/15 had a different neighborhood, and 7/10 rent records reported a yearly IPTU period that the page marked monthly.
- **Parking filter:** in search, `parkingMin` currently acts as an exact `parkingSpaces` match, so listings with more spaces can be missed. The intended minimum semantics require a code fix; use `byUrls` or omit the filter until then.
- **Approximate locations:** listings whose map point is approximate can omit `streetNumber`, `latitude`, and `longitude`, even though the page shows a neighborhood-level pin.
- **Truncated search titles:** some search `h2Tag` titles are truncated; `byUrls` returns the full portal title.
- **IPTU period ambiguity:** search mode follows the search API's `yearlyIptu`; the detail response (and `byUrls`) can mark the same value monthly. Use `byUrls` for detail-consistent periods.
- **Source caps:** VivaReal returns at most 30 listings per page. A run scans at most 5,000 rows before reporting a shortfall.
- **Photo quality:** images are requested at 1024×768 and may be unavailable if VivaReal removed them.
- **Brazil only:** there is no Colombian inventory, no ZAP Imóveis data, and no virtual tours or videos.
