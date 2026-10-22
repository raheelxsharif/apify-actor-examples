# Wallapop Listings by Keyword, Item & Seller

Scrape **Wallapop** — the large second-hand marketplace in Spain, Italy and Portugal. Search by
keyword or category, look up an item by its URL or ID, pull every active listing from a seller, or
read a seller's profile, ratings and reviews. The actor returns prices, condition, photos, location,
shipping, engagement counts and seller reputation as clean JSON. No login, no cookies, no API key,
and no proxy setup.

## What this actor does

- **Four modes:** `search`, `itemDetails`, `sellerListings`, `sellerProfile`.
- **Search by keyword or category:** multiple keywords, all 21 Wallapop categories and 99
  subcategories, with price, condition, posting-date, shipping, distance and sort filters.
- **Market-aware:** Spain, Italy, Portugal, France, Germany and the United Kingdom by coordinates,
  with EUR prices and country-local listings.
- **Item lookup by URL or ID:** full item detail including condition, attributes, images,
  reserves/sold status, views, favourites and conversations.
- **Seller tools:** every active listing from one or more sellers, plus profile stats, rating
  breakdown, public contact details, business legal data and recent reviews.
- **Empty fields are omitted:** a missing value is never returned as `null`, `""` or `[]`.

## Output per listing

Every listing record has `recordType: "listing"`.

- `itemId`, `title`, `description`, `itemUrl`, `webSlug`
- `price`, `currency`, `financedPrice`
- `condition`, `conditionText`
- `categoryId`, `categoryName`, `categoryPath`
- `brand`, `model`, `attributes` — Wallapop's structured item attributes
- `imageUrl`, `imageUrls`
- `city`, `country`, `postalCode`, `region`, `latitude`, `longitude`, `locationApproximate`
- `distanceM` — distance from the search centre, in metres (search mode)
- `publishedDate`, `modifiedDate` — legacy Unix timestamps, when Wallapop provides them
- `publishedAt`, `modifiedAt` — ISO 8601 versions
- `reserved`, `sold`, `expired`, `onHold`, `bumped`, `bumpType`
- `isRefurbished`, `hasWarranty`
- `shippable`, `shippingAllowed`
- `views`, `favorites`, `conversations`
- `seller`, `sellerId`, `sellerRating`, `sellerReviewCount`, `sellerVerified`, `sellerType`,
  `sellerIsTopProfile`, `sellerProfileUrl`, `sellerRegisteredAt`
- `sellerContact` — public address, phone, website and opening hours (business/pro sellers only)
- `sellerLegal` — public legal name, email, registration number and registry (business sellers only)
- `scrapedAt`

Which optional fields appear depends on the mode and on what Wallapop publishes:

- `search` includes location, dates, reserve/shipping flags and taxonomy out of the box. Condition,
  attributes, views, favourites and conversations require **Include item details**. Seller name,
  rating and verification require **Include seller details**.
- `itemDetails` includes condition, attributes, flags, views, favourites, conversations and seller
  reputation. `publishedDate`/`publishedAt` are not available for a single item.
- `sellerListings` includes condition, attributes, shipping, refurbished/warranty and seller fields.
  Location, dates, views and favourites require **Include item details**; published date is not
  available in this mode.

## Output per seller profile

`sellerProfile` returns one record per seller with `recordType: "seller"`.

- `sellerId`, `sellerName`, `sellerWebSlug`, `sellerProfileUrl`
- `sellerType`, `sellerVerified`, `sellerIsTopProfile`, `sellerIsFeatured`, `sellerBadge`
- `avatarUrl`, `registeredAt`, `sellerDescription`
- `rating`, `reviewCount`, `ratingBreakdown`
- `responseRate`, `scoringStars`, `activityLevel`, `validations`
- `activeListings`, `soldCount`, `purchases`, `sales`, `reportsReceived`, `successfulShipments`
- `city`, `country`, `postalCode`, `latitude`, `longitude`
- `website`, `phone`, `address`, `openingHours`
- `legalName`, `legalEmail`, `registrationNumber`, `registryName`, `selfCertification`
- `reviews[]` — reviews up to `maxReviews` (1–2000), newest first
- `scrapedAt`

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `search` | `search`, `itemDetails`, `sellerListings` or `sellerProfile` |
| `keywords` | array | `["iphone"]` | Search phrases; each is searched and results are merged without duplicates (mode=search) |
| `itemUrls` | array | – | Wallapop item URLs or alphanumeric item IDs such as `wzvg1nvq81zl`; the numeric suffix at the end of an item URL is not an item ID (mode=itemDetails) |
| `sellerIds` | array | – | Seller IDs, profile URLs or item URLs (mode=sellerListings / sellerProfile) |
| `country` | string | `ES` | Market centre: ES, IT, PT, FR, DE, GB or Custom coordinates |
| `latitude` | number | – | Optional search centre latitude |
| `longitude` | number | – | Optional search centre longitude |
| `category` | string | all | One of Wallapop's 21 top-level categories |
| `subcategory` | string | all | One of the 99 subcategories; can be used alone |
| `orderBy` | string | `most_relevance` | Relevance, newest, nearest, price low-to-high or price high-to-low |
| `minPrice` | integer | – | Minimum asking price in EUR |
| `maxPrice` | integer | – | Maximum asking price in EUR |
| `condition` | string | any | New, new in box, unopened, unworn, as good as new, good, fair or has given it all |
| `timeFilter` | string | any | Today, last 7 days or last 30 days |
| `shippableOnly` | boolean | `false` | Only listings whose seller allows shipping |
| `distanceKm` | integer | – | Maximum distance from the search centre (1–1000 km) |
| `includeItemDetails` | boolean | `false` | Add condition, attributes, views, favourites and conversations to listings |
| `includeSellerDetails` | boolean | `true` | Add seller name, rating, review count and verification to search results |
| `includeReviews` | boolean | `true` | Include recent reviews in seller profiles |
| `maxReviews` | integer | `20` | Maximum reviews per seller (1–2000); fetched in pages of 40 |
| `maxItems` | integer | `20` | Maximum records for search, itemDetails, sellerListings and sellerProfile (1–2000) |

### Example: keyword search in Madrid

```json
{
  "mode": "search",
  "keywords": ["iphone 14", "iphone 15"],
  "country": "ES",
  "orderBy": "newest",
  "minPrice": 200,
  "maxPrice": 700,
  "condition": "good",
  "includeItemDetails": true,
  "maxItems": 50
}
```

### Example: Italian category with a radius

```json
{
  "mode": "search",
  "country": "IT",
  "category": "12465",
  "orderBy": "closest",
  "distanceKm": 50,
  "shippableOnly": true,
  "maxItems": 100
}
```

### Example: one item by URL

```json
{
  "mode": "itemDetails",
  "itemUrls": ["https://es.wallapop.com/item/iphone-11-64gb-rojo-1315331517"]
}
```

### Example: a seller's catalogue and profile

```json
{
  "mode": "sellerListings",
  "sellerIds": ["https://es.wallapop.com/user/santiagob-407326113"],
  "includeItemDetails": true,
  "maxItems": 200
}
```

```json
{
  "mode": "sellerProfile",
  "sellerIds": ["pj9ygg2gvv6e"],
  "includeReviews": true,
  "maxReviews": 20
}
```

## Use cases

- **Price research:** compare asking prices for the same product across Spanish, Italian and
  Portuguese cities.
- **Resale sourcing:** follow a category with a price cap and spot under-priced listings early.
- **Seller vetting:** read seller ratings, review history and public business details before buying.
- **Catalogue monitoring:** export a shop's full active inventory to a spreadsheet on a schedule.
- **Demand analysis:** rank listings by views, favourites and conversations.
- **Marketplace analytics:** build a clean dataset of condition, attributes and geography.

## Limitations

- `publishedDate` is available in keyword/category search only. Wallapop does not expose a creation
  date on item or seller endpoints, so those modes omit it.
- Seller contact details and business legal data are returned only when the seller has made them
  public (business, shop or pro accounts).
- Reviews are fetched newest-first in pages of 40 until `maxReviews` (default 20, maximum 2,000) or
  the source stops. Verified full collections of 411/411, 1,226/1,226 and 1,714/1,714 reviews for
  sellers of those sizes; the practical ceiling is the seller's public review count.
- `sellerListings` returns active listings only. Sold items are not searchable; an item URL that is
  already sold still works through `itemDetails`.
- **Nearest** uses Wallapop's proximity ranking; the source does not guarantee a strict distance
  order. Sort on `distanceM` if you need exact ordering.
- `categoryId` is Wallapop's top-level taxonomy id, while `categoryName` is the leaf name and
  `categoryPath` is the full hierarchy. Names follow the source locale.
- Italy and Portugal have smaller catalogues. For the largest local result set use **Most relevant**
  or **Nearest**, and a distance that matches the country.

## FAQ

**Do I need a Wallapop account, cookies or an API key?**
No. The actor reads publicly accessible listing and seller data without authentication.

**Do I need to configure a proxy?**
No. The actor connects directly to Wallapop; no proxy configuration is required.

**Which countries are supported?**
Spain, Italy and Portugal are the primary markets. France, Germany and the United Kingdom also
return local listings when you choose their country or supply coordinates; inventory there is much
smaller.

**Why are some fields missing from a record?**
Wallapop omits data for some listings, and some fields are mode-specific. Empty fields are left out
rather than returned as `null`. Turn on **Include item details** for condition, attributes, views and
favourites.

**Why did I get fewer rows than `maxItems`?**
Wallapop exhausted the matching results, or the country filter removed cross-border listings. The
run status reports the requested and returned counts honestly. The actor never fabricates rows.

**Can I get sold items?**
Only if you already have the item URL. Keyword and category search return active listings, and
seller catalogues return active listings. `itemDetails` reports `sold: true` when Wallapop marks the
item sold.

**How many reviews can I get?**
The actor fetches reviews newest-first in pages of 40 until `maxReviews` is reached or Wallapop
has no older reviews. Set `maxReviews` up to 2,000. The run status reports a shortfall if the
source stops earlier.

**How fresh is the data?**
Every run reads the live marketplace. Values reflect Wallapop at the moment of collection.

**Is this affiliated with Wallapop?**
No. This is an independent third-party actor using publicly accessible marketplace data.
