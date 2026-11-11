# Willhaben Marketplace, Cars, Real Estate & Jobs Scraper

Willhaben.at is Austria's largest classifieds site — marketplace goods, used cars, apartments, houses, land and jobs all live in one place. This actor turns those listings into structured records: prices, locations, seller details, car specifications, property areas and energy data, job salaries, images and direct links. Search by keyword and filters, or paste any Willhaben search or listing URL. No login, cookies, API key or proxy setup is needed.

## What this actor does

- **Marketplace, cars, real estate and jobs in one actor** — choose a section or paste URLs from any of them.
- **Location-aware search** — filter by federal state, city, district or Austrian postcode.
- **Real filters, not post-filtering** — price, car make/model/year/mileage/fuel/transmission/body/drive/colour/power, property type/rooms/area/outdoor space/equipment, and job region/field/employment type/employer type.
- **Optional detail enrichment** — add full descriptions, seller contact details, equipment lists, energy certificates and rent breakdowns from each listing page.
- **Incremental monitoring** — on recurring runs return only new, changed, expired or reappeared listings/jobs.
- **Omit-empty output** — a field the source does not carry is left out, never sent as `null` or blank.

## Output per listing

Every record has `recordType: "listing"` and the `mode` that produced it.

**Identity and pricing**

- `adId`, `uuid`, `heading`, `description`, `descriptionSnippet`
- `price`, `currency`, `priceDisplay`, `oldPrice`, `priceReductionDate`, `pricePerSquareMeter`
- `adUrl`, `categoryId`, `locationId`

**Location**

- `location`, `district`, `state`, `postcode`, `address`, `street`, `municipality`, `postalName`, `country`
- `latitude`, `longitude`

**Dates, status and seller**

- `publishedDate`, `changedDate`, `endDate`, `firstPublishedDate`
- `status`, `isPrivate`, `sellerType`
- `seller`, `sellerId`, `sellerContactName`, `sellerPhone`, `sellerWebsite`, `sellerLogoUrl`

**Media and extras**

- `imageUrl`, `imageUrls`, `imageCount`, `floorPlanUrls`
- `rawAttributes` (when "Include raw attributes" is on)
- `scrapedAt`

### Cars & vehicles

- `carMake`, `carModel`, `carMakeId`, `carModelId`, `carModelDetail`, `vehicleCategory`
- `year`, `registrationMonth`, `mileageKm`, `fuelType`, `transmission`
- `engineDisplacementCc`, `powerKw`, `consumptionLper100km`, `co2EmissionsGperKm`, `batteryCapacityKwh`
- `bodyType`, `condition`, `driveType`, `color`, `doors`, `seats`, `owners`
- `warranty`, `inspection`, `inspectionValidUntil`, `equipment[]`

### Real estate

- `propertyType`, `propertyTypeId`, `rooms`, `roomsCount`, `livingAreaSqm`, `usableAreaSqm`, `plotAreaSqm`
- `floor`, `floorSurface`, `freeAreaType`, `freeAreaSqm`, `heating`, `buildingCondition`, `constructionYear`
- `energyClassHwb`, `energyDemandHwb`, `energyClassFgee`, `energyDemandFgee`
- `availableDate`, `availableNow`, `ownershipType`, `rentalTerm`
- `rentPerMonth`, `rentNet`, `rentVat`, `totalEncumbrance`, `deposit`, `additionalCosts`, `monthlyCosts`
- `equipment[]`

## Output per job

Job records have `recordType: "job"`.

- `jobId`, `jobTitle`, `description`, `jobUrl`
- `companyName`, `companyId`, `companySlug`, `companyUid`, `companyAddress`, `companyLogoUrl`
- `position`, `salary`, `salaryTimeFrame`, `employmentTime`, `employmentModes[]`
- `locations[]` — objects with `name`, `federalState`, `country`
- `publishedDate`, `modifiedDate`, `expiryDate`
- `isExpired`, `isTopJob`, `overpay`, `brandNew`, `internalApplication`
- `contactName`, `contactEmail`, `languageSkills[]`, `professionalExperience`
- `mode`, `scrapedAt`

Add `includeDetails` to fill full descriptions, expiry dates, company addresses, contact details and language skills where the source publishes them. Contact email is only present on some job ads.

## Incremental output

When `incrementalMode` is on, each record also has:

- `changeType` — `NEW`, `UPDATED`, `UNCHANGED`, `REAPPEARED` or `EXPIRED`
- `changedFields[]` — tracked fields that changed on an `UPDATED` record
- `firstSeenAt`, `lastSeenAt`, `previousPrice`
- `emitUnchanged` / `emitExpired` control whether unchanged or missing records are included

The first run returns everything as `NEW`. Later runs compare against the saved state for the same `stateKey` (or the same filters when `stateKey` is empty).

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `searchItems` | `searchItems`, `searchCars`, `searchRealEstate`, `searchJobs` or `byUrls` |
| `query` | string | – | Keyword to search for |
| `startUrls` | array | – | Willhaben search, category, listing, job search or job URLs (mode `byUrls`) |
| `state` | string | – | Austrian federal state or other countries |
| `location` | string | – | City, district or postcode; for jobs it is the job location |
| `radiusKm` | integer | – | Job search radius, 1–100 km |
| `sortBy` | string | `relevance` | `relevance`, `newest`, `priceAsc`, `priceDesc`, `distance` |
| `priceMin`, `priceMax` | integer | – | Price range in EUR (ignored for jobs) |
| `maxItems` | integer | `20` | Maximum listings or jobs (1–50,000) |
| `includeDetails` | boolean | `false` | Fetch full descriptions, contact/seller and extra detail fields |
| `includeRawAttributes` | boolean | `false` | Add every source attribute as `rawAttributes` |
| `sellerType` | string | – | `private` or `dealer` (marketplace items and cars; ignored for real estate and jobs) |
| `vehicleCategory` | string | `cars` | `cars`, `motorcycles`, `caravans` or `commercial` |
| `carMake` | string | – | Car manufacturer |
| `carModel` | string | – | Model name; requires `carMake` |
| `yearFrom`, `yearTo` | integer | – | First-registration year range |
| `mileageFrom`, `mileageTo` | integer | – | Mileage range in km |
| `fuelType` | string | – | Petrol, diesel, electric, gas, hybrid or hydrogen |
| `transmission` | string | – | Automatic or manual |
| `bodyType` | string | – | Convertible, compact, estate, saloon, SUV, etc. |
| `carCondition` | string | – | Used, new, classic, demonstration car, etc. |
| `driveType` | string | – | All-wheel, rear-wheel or front-wheel drive |
| `color` | string | – | Exterior colour |
| `powerFrom`, `powerTo` | integer | – | Engine power range in kW |
| `realEstateCategory` | string | `rentApartment` | Rent/buy apartment or house, land, new-build, commercial, holiday or other |
| `propertySubtype` | string | – | Penthouse, detached house, building plot, etc. |
| `rooms` | string | – | Room-count bucket (`1`, `2`, … `10+`, not specified) |
| `livingAreaFrom`, `livingAreaTo` | integer | – | Living area range in m² |
| `plotAreaFrom`, `plotAreaTo` | integer | – | Plot area range in m² |
| `freeAreaType` | string | – | Balcony, terrace, garden, loggia, winter garden or roof terrace |
| `equipment` | string | – | Lift, cellar, parking, built-in kitchen, etc. |
| `availableNow` | boolean | `false` | Only properties available immediately |
| `jobRegion` | string | – | Austrian state, Austria, Germany or Switzerland |
| `employmentMode` | string | – | Full-time, part-time, freelance or marginal |
| `jobPosition` | string | – | Management, team lead, employee, apprenticeship, internship, etc. |
| `jobOperationArea` | string | – | Job field such as IT, care, finance, sales or trades |
| `jobCompanyType` | string | – | Staffing agency or direct employer |
| `jobTimeLimit` | string | – | Any time, last 24 hours, last 72 hours or last week |
| `incrementalMode` | boolean | `false` | Return only new/changed records on recurring runs |
| `stateKey` | string | – | Stable name for the monitoring campaign |
| `emitUnchanged` | boolean | `false` | Include unchanged records |
| `emitExpired` | boolean | `false` | Include records that disappeared after a complete scan |

### Example: marketplace furniture

```json
{
  "mode": "searchItems",
  "query": "sofa",
  "state": "Wien",
  "priceMax": 800,
  "sortBy": "priceAsc",
  "maxItems": 100
}
```

### Example: used BMW with detail enrichment

```json
{
  "mode": "searchCars",
  "carMake": "1005",
  "carModel": "3er-Reihe",
  "yearFrom": 2018,
  "mileageTo": 120000,
  "fuelType": "100003",
  "priceMax": 30000,
  "includeDetails": true,
  "maxItems": 50
}
```

### Example: Vienna rental apartments

```json
{
  "mode": "searchRealEstate",
  "realEstateCategory": "rentApartment",
  "state": "Wien",
  "rooms": "2X2",
  "livingAreaFrom": 55,
  "priceMax": 1400,
  "maxItems": 100
}
```

### Example: chef jobs in Vienna

```json
{
  "mode": "searchJobs",
  "query": "Koch",
  "jobRegion": "14486",
  "employmentMode": "110",
  "jobTimeLimit": "last_week",
  "includeDetails": true,
  "maxItems": 50
}
```

### Example: paste search and listing URLs

```json
{
  "mode": "byUrls",
  "startUrls": [
    { "url": "https://www.willhaben.at/iad/immobilien/mietwohnungen/wien?PRICE_TO=1200" },
    { "url": "https://www.willhaben.at/iad/gebrauchtwagen/d/auto/bmw-3er-reihe-320d-1234567890/" },
    { "url": "https://www.willhaben.at/jobs/suche?keyword=koch&region=14486" }
  ],
  "maxItems": 30
}
```

## Use cases

- **Car dealers and buyers** — track make/model/year/mileage/price across Austria and compare private vs dealer offers.
- **Real-estate analysts** — build €/m², rent, energy-class and availability datasets by district.
- **Property hunters** — monitor Vienna or Graz rentals and new-build projects with incremental runs.
- **Recruiters and job analysts** — export job ads with salary, company, employment type and location.
- **Resellers** — watch marketplace categories for new listings and price reductions.
- **Market researchers** — collect cross-section supply data for Austrian goods and property.

## FAQ

**Is this affiliated with Willhaben?**
No. This actor reads publicly visible listings and has no relationship with Willhaben.at.

**Do I need an account, cookies or an API key?**
No. Listings, search pages and job ads are read without logging in.

**Do I need to configure a proxy?**
No. The actor chooses a working connection automatically. Just fill in the input and run it.

**Why are some fields missing on some records?**
Sellers do not always publish every attribute. A field the source does not carry is omitted, never sent as `null` or a blank value.

**Why is car mileage or first registration missing on some cars?**
Some sellers leave those fields empty. The actor returns the data the listing actually contains.

**What does "Fetch detail pages" add?**
Full descriptions, seller/contact details, equipment lists, energy certificates, rent breakdowns, floor plans and extra car specification fields. It is optional because it makes the run slower.

**How many records can one run return?**
Up to `maxItems`, which can be set to 50,000. Willhaben serves at most 199,800 records per search query; for larger jobs, split by state or price range and combine the results.

**Can I run this on a schedule and only get new listings?**
Yes. Turn on `incrementalMode`; the first run establishes the baseline and later runs return new, changed, expired or reappeared records according to `emitUnchanged` and `emitExpired`.

**Do price and seller-type filters work for jobs and real estate?**
Job ads have no listing price, so price filters are ignored for jobs. Willhaben does not apply seller-type filtering to real estate or jobs; those filters are ignored and the run reports it in its status message.

**Are image and listing links usable outside Apify?**
Yes. Image URLs and listing/job links are public absolute URLs.
