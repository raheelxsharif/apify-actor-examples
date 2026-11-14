# Domain.com.au Property Listings & Agent Data Extractor

Domain.com.au is one of Australia's two major property portals, listing homes for sale, for rent and recently sold across every state. Point this actor at a Domain search URL (a suburb, region or `/sale/`, `/rent/`, `/sold-listings/` path) or at an individual property's own URL, and it returns each listing as a flat, structured record — no browsing through result pages or opening listings one by one.

## What this actor does

- Walks Domain.com.au search result pages (sale, rent or sold) and paginates automatically until it has collected `maxItems` listings
- Also accepts a direct Domain property URL, which is scraped as a single listing with extra detail-page fields (agents, floor plans, school catchments, price guide, auction/sale details)
- Parses structured and contact fields such as agents, agency branding, inspection times and virtual tours when the listing publishes them
- Infers the listing type (`sale`, `rent`, `sold`) from the search URL path
- Empty fields are omitted from records — a field only appears when Domain actually published that data for the listing

## Output per listing

Core fields present for both search-page and direct-URL listings:

- `type` — always `domain_listing`
- `id` — Domain's internal listing ID
- `url` — canonical Domain.com.au listing URL
- `title` — short address line, e.g. `107/104 Fairway Drive, Norwest`
- `price` — numeric price, when Domain exposes an exact figure
- `priceLabel` — price as displayed, e.g. `$1,000,000`, `Auction`, `Contact agent`
- `propertyType` — `House`, `ApartmentUnitFlat`, `Townhouse`, `Land`, etc.
- `bedrooms`, `bathrooms`, `carspaces` — integer counts
- `address`, `suburb`, `state`, `postcode`
- `latitude`, `longitude`
- `images` — array of photo URLs
- `inspections` — array of open-home inspection times (open/close)
- `agent` — `{name, photo}` of the primary listing agent
- `agencyName`, `agencyColor`, `agencyLogo` — agency branding
- `features` — amenity/feature tags (e.g. "Air Conditioning")
- `tags` (or `tag` for a single badge) — marketing badge(s) such as "New home" or "Just listed"
- `projectName` — development name, for off-the-plan listings
- `listingType` — `sale`, `rent` or `sold`, inferred from the source URL
- `scrapedAt` — UTC timestamp when the record was collected

Additional fields that only appear when a listing is scraped from its own direct URL (not from a search page):

- `agents` — full agent list, each with `name`, `photo`, `phone`, `mobile`, `email`, `profileUrl`
- `agentPhone`, `agencyPhone`, `agencyProfileUrl`, `agencyWebsite`, `agencyBanner`
- `floorplans`, `videos`, `virtualTourUrl`, `matterportInterior`, `matterportExterior`
- `projectUrl` — link to the off-the-plan project page, for project listings
- `landArea`, `buildingArea`, `energyEfficiencyRating`
- `headline`, `tagline`, `descriptionHtml`
- `listedAt`, `createdOn`, `modifiedOn`
- `salesMethod`, `listingStatus`, `soldAt`, `soldSaleMethod`
- `auctionDate`, `auctionLocation`
- `priceGuide` — `{from, to}` estimated price range, when Domain publishes one
- `structuredFeatures` — list of `{name, category}` property features
- `schoolCatchment` — nearby zoned schools, each with `name`, `educationLevel`, `type`, `gender`, `year`
- `propertyReference` — Domain's internal property identifier

If a run produces no listings at all (wrong region slug, or no matches), the actor emits a single `domain_blocked` record instead: `type: "domain_blocked"`, `reason` (explanation text), `requestedUrls` (the `searchUrls` you passed in), and `scrapedAt` — so the run still completes with output.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `searchUrls` | array | `["https://www.domain.com.au/sale/sydney-region-nsw/"]` | Domain.com.au search URLs (`/sale/...`, `/rent/...`, `/sold-listings/...`) or direct listing URLs |
| `maxItems` | integer | `3` | Maximum property listings to collect per run (1–500) |
| `proxyConfiguration` | object | `{"useApifyProxy": false}` | Optional Apify proxy; Domain.com.au is reachable without one, so this is disabled by default |

### Example: sale listings in a region

```json
{
  "searchUrls": ["https://www.domain.com.au/sale/sydney-region-nsw/"],
  "maxItems": 50
}
```

### Example: rentals in a specific suburb

```json
{
  "searchUrls": ["https://www.domain.com.au/rent/melbourne-region-vic/"],
  "maxItems": 100
}
```

### Example: enrich a single property page

```json
{
  "searchUrls": [
    "https://www.domain.com.au/107-104-fairway-drive-norwest-nsw-2153-2020305174"
  ],
  "maxItems": 1
}
```

## Use cases

- **Comparable sales (comps)** — pull `/sold-listings/` pages for a suburb to see recent sale prices against bedroom/bathroom counts
- **Rental market monitoring** — track asking rents across a region's `/rent/` listings over time
- **Agent lead generation** — collect agent and agency contact details from listings in a target suburb
- **Buyer alerts** — re-scrape a saved search URL on a schedule to catch newly listed properties
- **Listing enrichment** — pass a single property URL to pull the deeper detail-page fields (floor plans, school catchments, price guide) for a shortlist of properties

## FAQ

**Do I need a proxy?** No. Domain.com.au pages load fine from standard datacenter IPs, so the `proxyConfiguration` input is off by default.

**What's the difference between a search URL and a direct listing URL?** A search URL (e.g. ending in `/sale/sydney-region-nsw/`) returns many listings up to `maxItems`. A direct listing URL returns exactly that one property, but with extra detail-page fields like agents, floor plans and school catchments that search-result cards don't carry.

**Can I scrape sold listings, not just active ones?** Yes — any URL under `/sold-listings/` works the same way as `/sale/` or `/rent/` URLs, and `listingType` will be set to `sold`.

**Why do some listings have no `price`?** Many Australian listings are marketed as "Auction" or "Contact agent" rather than a fixed price. In that case `priceLabel` carries the display text and `price` is omitted.

**Why is `schoolCatchment` or `floorplans` missing from a listing?** Those fields only come from the richer detail page and only when Domain actually publishes that data for the property; scraping from a search URL alone won't populate them.

**What happens if my search URL has no matches?** The actor still finishes successfully and pushes a single `domain_blocked` record explaining that nothing was found, so your downstream pipeline always sees output.

**Is this affiliated with Domain.com.au?** No. This is an independent, third-party actor that reads Domain.com.au's public listing pages; it is not operated by or affiliated with Domain Group.
