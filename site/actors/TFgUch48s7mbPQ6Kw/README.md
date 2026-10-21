# Zillow Distressed Property Finder by City or ZIP

Zillow keeps its foreclosure, pre-foreclosure, pre-market REO and auction inventory inside its regular map search. This actor applies those distress filters server-side for you and returns structured records for any US city, ZIP code, neighborhood or county, with optional per-listing detail enrichment: listing agent, phone, brokerage, photos, description, year built, HOA and per-row distress flags.

## What this actor does

- Searches one or more locations: city ("Austin, TX"), ZIP ("78701"), neighborhood ("Hyde Park, Austin, TX") or county ("Travis County, TX").
- Also accepts Zillow search URLs you already have; the distress filter is layered on top of the URL's own filters.
- Restricts results to the distress categories you pick: foreclosures (including bank-owned), pre-foreclosures, pre-market foreclosed (REO) and auctions.
- Applies Zillow's own filters server-side: price range, minimum beds/baths, home types, days on Zillow and sort order.
- Pages through Zillow's result pages up to its own limit, with no browser and no cookies required.
- Optionally opens each listing's detail page for the listing agent, photos, description, year built, HOA and authoritative distress flags.
- Fields Zillow doesn't publish for a listing (no price, no Zestimate, no photos) are left out of that record rather than filled with a placeholder.

## Output per listing

| Field | Description |
|---|---|
| `zpid` | Zillow property ID. |
| `url` | Zillow property detail page URL. |
| `distressType` | `foreclosure`, `bankOwned`, `preForeclosure`, `preMarketForeclosure`, `auction` or `comingSoon`. |
| `isForeclosure` / `isBankOwned` / `isPreForeclosure` / `isAuction` / `isComingSoon` | Per-listing distress flags as Zillow marks them. |
| `homeStatus` / `statusText` | Raw Zillow status and its display label. |
| `address` / `city` / `state` / `zipCode` | Property address components. |
| `price` | Listing price, in USD. |
| `zestimate` / `rentZestimate` | Zillow's value and rent estimates, in USD. |
| `taxAssessedValue` | Tax-assessed value, in USD. |
| `beds` / `baths` / `sqft` | Bedroom and bathroom counts, and living area in square feet. |
| `lotAreaValue` / `lotAreaUnit` | Lot size and its unit (`sqft` or `acres`). |
| `propertyType` / `yearBuilt` | Property type (e.g. `SINGLE_FAMILY`) and construction year. |
| `latitude` / `longitude` | Property coordinates. |
| `daysOnZillow` / `listedDate` | Days on Zillow and the date the listing was posted. |
| `listingAgentName` / `listingAgentPhone` / `brokerName` / `mlsName` / `mlsId` | Listing attribution from the detail page. |
| `photos` / `photoCount` | Property photo URLs (largest size, Zillow photo CDN only) and how many photos Zillow shows. |
| `description` | Listing description text. |
| `monthlyHoaFee` / `pricePerSquareFoot` | HOA fee and price per square foot when shown. |
| `postingUrl` | External listing or auction page when Zillow provides one (e.g. an auction.com detail link). |
| `auctionDate` / `auctionDateTime` | Auction date and sale start date-time, read from the linked auctioneer page. |
| `startingBid` / `principalLoanAmount` | Published starting bid and original loan principal when the auctioneer shows them. |
| `occupancyStatus` | Occupancy reported by the auctioneer (e.g. `OCCUPIED`, `VACANT`). |
| `lender` / `servicer` | Beneficiary (lender) and loan servicer names. |
| `foreclosureAttorney` / `foreclosureAttorneyPhone` | Foreclosure attorney / trustee firm and its phone. |
| `buyerPremiumAvailable` | Whether the auction charges a buyer's premium. |
| `auctionEventTitle` | The auctioneer's sale event title. |
| `lastSoldPrice` / `lastSoldDate` | Most recent recorded sale price and date, when Zillow publishes them. |
| `comingSoonDate` | The date a coming-soon listing is scheduled to go live. |
| `scrapedAt` | ISO 8601 timestamp of when this actor fetched the record. |

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `locations` | Array of strings | – | Cities, ZIP codes, neighborhoods or counties — one entry each. |
| `startUrls` | Array of URLs | – | Zillow search pages; the distress filter is applied on top. |
| `distressTypes` | Array (select) | all four | `foreclosure`, `preForeclosure`, `preMarketForeclosure`, `auction`; legacy `bankOwned` (REO-only) is also accepted. Leave empty for all. |
| `priceMin` / `priceMax` | Integer | – | Listing price bounds in USD. |
| `bedsMin` / `bathsMin` | Number | – | Minimum bedrooms / bathrooms. |
| `homeTypes` | Array (select) | all | `singleFamily`, `condo`, `townhouse`, `multiFamily`, `apartment`, `manufactured`, `lotLand`. |
| `daysOnZillow` | Select | `any` | Only listings seen within the window (1 day … 36 months). |
| `sortBy` | Select | `relevance` | Relevance, newest, price, beds, baths, size, lot or Zestimate. |
| `includeDetails` | Boolean | `true` | Open each listing's detail page for agent, photos, description and year built. |
| `includeAuctionDetails` | Boolean | `false` | Read the linked auctioneer page (auction.com only) for the auction schedule, starting bid, occupancy and lender/attorney fields. One extra request per linked auction; requires `includeDetails`. |
| `maxItems` | Integer | `100` | Maximum listings across all locations and URLs (1–5000). |

Provide at least one `locations` entry or one `startUrls` entry.

### Example: foreclosures and auctions in one city

```json
{
    "locations": ["Phoenix, AZ"],
    "distressTypes": ["foreclosure", "auction"],
    "priceMax": 400000,
    "includeDetails": true,
    "maxItems": 100
}
```

### Example: bank-owned homes by ZIP code

```json
{
    "locations": ["85001", "85032"],
    "distressTypes": ["preMarketForeclosure"],
    "includeDetails": true,
    "maxItems": 200
}
```

### Example: reuse a saved Zillow search URL

```json
{
    "startUrls": [{"url": "https://www.zillow.com/austin-tx/foreclosures/"}],
    "maxItems": 300
}
```

## Use cases

- **Distressed-inventory investing** — track foreclosure and pre-foreclosure properties before they hit the broader market.
- **Wholesaling** — build a pre-foreclosure outreach list for a ZIP or neighborhood.
- **Auction prep** — research properties scheduled for a foreclosure auction ahead of time.
- **REO acquisition** — surface pre-market bank-owned properties for buy-and-hold portfolios.
- **Market intelligence** — quantify distress rates by city, ZIP or county over time.

## FAQ

**What's the difference between the distress types?**
`foreclosure` is an active foreclosure listing for sale (Zillow's foreclosure filter also includes bank-owned homes); `bankOwned` marks a home the lender already owns (REO); `preForeclosure` is a pre-market record early in the foreclosure process, not yet listed; `preMarketForeclosure` is a pre-market record the bank has already taken back; `auction` is a property scheduled for a foreclosure auction.

**How many results can I get per run?**
Up to 5,000 via `maxItems`. Zillow itself caps any single search at 20 result pages (about 820 listings). If a market hits that cap, split it into several `locations` — by home type, ZIP or neighborhood — and pass them together; the slices are disjoint, so their union covers the full inventory (verified: Los Angeles distressed inventory = 1,349 listings; its seven home-type slices sum to exactly 1,349).

**Do I need proxies or cookies?**
No. The actor routes its requests automatically and needs no account, cookie or API key from you.

**Why do some listings have no price or Zestimate?**
Zillow genuinely doesn't show a price or estimate for many pre-market distressed records. Those fields are omitted from the record instead of being filled with a placeholder.

**Does this include auction dates, unpaid balances or lender details?**
Zillow's own pages don't carry them, but auction listings link out to the auctioneer (`postingUrl`). Turn on `includeAuctionDetails` and the actor also reads the linked auction.com page, adding the auction date and sale time, occupancy, starting bid, loan principal, lender, servicer and foreclosure attorney (each when the auctioneer publishes it). Roughly two thirds of sampled Zillow auctions link to auction.com; other auctioneers and unpaid balances are not covered.

**Is this affiliated with Zillow?**
No. This actor is an independent tool that reads publicly visible Zillow search results and is not affiliated with or endorsed by Zillow.
