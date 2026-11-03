# Brazil Rental & Sale Listings by City (QuintoAndar)

QuintoAndar is one of Brazil's largest home rental and sale marketplaces, covering São Paulo, Rio de Janeiro, Belo Horizonte and dozens of other metro areas. This actor searches its public listings by city or neighborhood and returns structured property records: transaction prices, monthly costs, size, rooms, address, amenities, photos and optional detail-page enrichment. No account, cookie or proxy setup is needed.

## What this actor does

- **Three modes:** search rentals (`rent`), search properties for sale (`buy`), or load specific listings by URL or numeric ID (`byUrls`)
- **99 locations**, with optional neighborhood search inside any city
- **Server-side filters:** property type, price (base rent, total monthly cost or sale price), area, bedrooms, bathrooms, parking, suites, furnished, pets, near subway, amenities, building facilities, availability and sort order (`cheapest`/`expensive` use total monthly cost; `highestRent` is the rental-return order)
- **Large result sets:** up to 1,000 listings per query and up to 5,000 per run through automatic price-slice partitioning
- **Optional detail enrichment:** description, separate condo fee and IPTU, ZIP code, state, coordinates, full photo gallery with captions, construction year, floor range and project data
- **Empty fields are omitted** from each record instead of being returned as null

## Output per property

- `propertyId` — QuintoAndar property ID
- `title` — composed title with property type, bedrooms, neighborhood and city
- `propertyType` — Apartamento, Casa, CasaCondominio or StudioOuKitchenette
- `rentPrice` — base monthly rent in BRL
- `salePrice` — sale price in BRL
- `totalMonthlyCost` — rent plus condo fee and IPTU, in BRL
- `condoAndIptu` — combined condo fee and IPTU, in BRL
- `condoFee` — monthly condo fee in BRL (with detail enrichment)
- `iptu` — monthly IPTU in BRL (with detail enrichment)
- `iptuPeriod` — always `monthly` when `iptu` is present
- `currency` — always `BRL`
- `areaSqM` — usable area in square metres
- `bedrooms`, `bathrooms`, `suites`, `parkingSpots` — room counts; real zero values are kept
- `street`, `neighborhood`, `region`, `city` — address fields
- `state`, `stateCode`, `zipCode` — state and postal code (with detail enrichment)
- `latitude`, `longitude` — map coordinates (with detail enrichment)
- `amenities[]` — unit feature codes, e.g. `VARANDA`, `AR_CONDICIONADO`
- `amenitiesText[]` — human-readable amenity labels (with detail enrichment)
- `installations[]` — building facility codes, e.g. `PISCINA`, `ACADEMIA`
- `installationsText[]` — human-readable facility labels (with detail enrichment)
- `photos[]` — every property photo as a working image URL
- `photoCaptions[]` — captions parallel to `photos[]` when every photo has one
- `imageUrl` — primary property image URL
- `isFurnished`, `forSale`, `forRent`, `acceptsPets`, `nearSubway` — listing flags
- `yieldRate`, `yieldStrategy` — rental yield and its calculation basis, when published
- `visitStatus` — current visit availability
- `lastPublishedDate`, `firstPublishedDate` — publication timestamps
- `listingTags[]`, `specialConditions[]` — listing tags and exclusivity-style conditions
- `isPrimaryMarket`, `developmentId`, `developmentName`, `constructionStatus`, `forecastDeliveryDate` — new-development fields
- `condoName`, `condoSlug` — condominium/project name and slug
- `constructionYear`, `floorMin`, `floorMax` — building year and floor range
- `shortDescription`, `description` — short and long listing text
- `propertyUrl` — canonical QuintoAndar listing page
- `mode` — transaction type of the record (`rent`, `buy` or `byUrls`)
- `recordType` — always `property`
- `scrapedAt` — ISO 8601 timestamp of when the listing was scraped

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `rent` | `rent`, `buy` or `byUrls` |
| `city` | string | `sao-paulo-sp-brasil` | One of 99 QuintoAndar locations (mode=rent or buy) |
| `neighborhood` | string | – | Optional neighborhood inside the city, e.g. Pinheiros (mode=rent or buy) |
| `propertyType` | string | `ANY` | `ANY`, `APARTMENT`, `HOUSE`, `HOUSE_CONDO` or `STUDIO` |
| `priceBasis` | string | `rent` | `rent` or `total` (mode=rent); ignored for buy |
| `priceMin` / `priceMax` | integer | – | Price range in BRL |
| `areaMin` / `areaMax` | integer | – | Usable area range in m² |
| `bedroomsMin` / `bedroomsMax` | integer | – | Bedroom range |
| `bathroomsMin` / `bathroomsMax` | integer | – | Bathroom range |
| `parkingMin` / `parkingMax` | integer | – | Parking space range |
| `suitesMin` / `suitesMax` | integer | – | Suite range |
| `furnished` | string | `any` | `any`, `yes` or `no` |
| `acceptsPets` | string | `any` | `any` or `yes` |
| `nearSubway` | string | `any` | `any` or `yes` |
| `amenities` | array | – | Required unit feature codes |
| `installations` | array | – | Required building facility codes |
| `availability` | string | `any` | `any`, `immediate` or `soon` (mode=rent) |
| `sortBy` | string | `relevance` | `relevance`, `newest`, `cheapest`, `expensive` or `highestRent` (highest rental return / yield) |
| `developmentsOnly` | boolean | `false` | Show only new developments (mode=buy) |
| `exactCityOnly` | boolean | `false` | Drop listings from neighbouring municipalities |
| `includeDetails` | boolean | `true` | Load description, condo/IPTU split, ZIP, coordinates and full captions |
| `listingUrls` | array | – | Listing URLs or numeric IDs (mode=byUrls) |
| `maxItems` | integer | `500` | Hard cap, 1–5,000 |

### Example: rentals in São Paulo

```json
{
  "mode": "rent",
  "city": "sao-paulo-sp-brasil",
  "maxItems": 200,
  "includeDetails": false
}
```

### Example: family-sized apartments for sale in Rio de Janeiro

```json
{
  "mode": "buy",
  "city": "rio-de-janeiro-rj-brasil",
  "propertyType": "APARTMENT",
  "bedroomsMin": 3,
  "priceMin": 800000,
  "priceMax": 2500000,
  "sortBy": "newest",
  "maxItems": 100
}
```

### Example: pet-friendly rentals in Pinheiros with a pool

```json
{
  "mode": "rent",
  "city": "sao-paulo-sp-brasil",
  "neighborhood": "Pinheiros",
  "acceptsPets": "yes",
  "installations": ["PISCINA"],
  "priceMax": 6000,
  "maxItems": 50
}
```

### Example: load specific listings

```json
{
  "mode": "byUrls",
  "listingUrls": [
    "https://www.quintoandar.com.br/imovel/893977588",
    "895686771"
  ]
}
```

## Use cases

- **Rental budgeting** — compare base rent, condo fee, IPTU and total monthly cost across neighborhoods
- **Relocation research** — scan a new city or a single neighborhood before an apartment hunt
- **Market analysis** — measure price per square metre by city, property type and size band
- **Investment screening** — compare sale prices and published rental yields for similar properties
- **Portfolio monitoring** — re-run the same city and filters to spot new listings and price changes
- **Project tracking** — follow new developments with construction status and delivery forecasts

## FAQ

### Is this actor affiliated with QuintoAndar?

No. It is an independent, third-party tool that reads publicly visible QuintoAndar listings. It is not operated by or affiliated with QuintoAndar.

### Do I need a QuintoAndar account, cookies or a proxy?

No. All supported modes work without an account or cookies, and no proxy setup is required.

### What is the difference between `rentPrice`, `totalMonthlyCost` and `condoAndIptu`?

`rentPrice` is the base rent. `condoAndIptu` is the combined condo fee and IPTU. `totalMonthlyCost` adds those extras to the rent, matching the total QuintoAndar displays for rent listings.

### Why does a rental listing sometimes have a `salePrice`?

Some owners list the same property for both rent and sale. When QuintoAndar publishes both prices, both fields are included.

### How many listings can one run return?

A single location query yields at most 1,000 listings, which is QuintoAndar's own per-query limit. Runs above 1,000 automatically split the price range into slices and merge the results, up to `maxItems` (5,000). If the source has fewer matching listings, the run stops honestly with the real count.

### Why do results include cities other than the one I selected?

QuintoAndar city searches cover the surrounding metro region, so a São Paulo search can include Mauá or Guarulhos. Set `exactCityOnly` to keep only the city you selected.

### Why are `suites`, `iptu`, `description` or coordinates missing on some records?

Those fields are conditional in the source: some listings do not publish a suite count, IPTU value or description. They are omitted rather than filled with a placeholder. Turning on `includeDetails` fills them whenever the listing page has them.

### Can I get the agent's name, phone or email?

No. QuintoAndar does not expose listing-agent contact details on public pages, so the actor cannot return them.

### What do the amenity and facility codes mean?

They are QuintoAndar's own codes, such as `VARANDA` (balcony), `AR_CONDICIONADO` (air conditioning), `PISCINA` (pool) and `ACADEMIA` (gym). With `includeDetails` enabled, the actor also returns the Portuguese labels in `amenitiesText` and `installationsText`.

## Limitations

- **1,000 listings per location query.** Larger requests are split by price, but a slice with more than 1,000 listings at the same price can still be capped.
- **Source inventory is finite.** A run cannot return more listings than QuintoAndar currently has for the chosen location and filters.
- **City searches include metro municipalities.** Use `exactCityOnly` when you need the city boundary only.
- **No agent or advertiser contact details.** QuintoAndar does not publish them.
- **No total area or land area.** QuintoAndar exposes usable area (`areaSqM`) only.
- **Some fields are conditional.** Suites, IPTU, descriptions, construction year and project data appear only when the listing provides them.
- **Price-slice partitioning changes ordering.** Runs above 1,000 are assembled from multiple price slices, so the final order differs from a single sorted page.
