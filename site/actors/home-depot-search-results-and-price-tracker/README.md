# Home Depot Products, Prices & Store Stock

Scrape Home Depot products from a keyword search, a category page or a list of item IDs. Every row can include the store-local price, pickup and delivery availability, aisle and bay, full specifications, media, variants, and optional reviews and Q&A.

## What this actor does

- **Three ways to start** — search by keyword, browse a whole category, or look up specific item IDs and product URLs.
- **Store-local pricing and stock** — pick a Home Depot store (or a ZIP to find the nearest one) and read that store's price and shelf pickup quantity.
- **Pickup and delivery kept separate** — shelf pickup quantity, ship-to-store availability, and online/delivery quantity are separate fields and are never mixed.
- **Full product detail** — description, highlights, specification groups, key features, dimensions, UPC/GTIN, all images, videos, 360/AR links, bundle contents and variant swatches.
- **Optional reviews and Q&A** — add a capped set of customer reviews with rating breakdown, and customer questions with their answers.
- **Filters and pagination** — price, brand, rating, sale, clearance, stock, sponsored and title filters, plus multi-page collection with duplicate removal.
- **Empty fields are omitted** from each record instead of being returned as null or blank.

## Output per product

**Identity** — `recordType`, `itemId`, `parentId`, `url`, `title`, `brand`, `modelNumber`, `storeSku`, `productType`, `upc`, `gtin13`, `brandUrl`, `collection`, `scrapedAt`

**Pricing** — `price`, `originalPrice`, `currency`, `discountAmount`, `discountPercent`, `unitOfMeasure`, `unitPrice`, `bulkPrice`, `clearancePrice`, `isSpecialBuy`, `isClearance`, `onPromotion`, `promotionTag`, `savingsCenter`, `promotionStartsAt`, `promotionEndsAt`, `conditionalPromotions`

**Ratings and reviews** — `rating`, `reviewCount`, `ratingDistribution`, `recommendedCount`, `notRecommendedCount`, `reviews[]` with text, author, date, verified purchase, photos, responses, and source

**Media and content** — `imageUrl`, `images[]`, `videos[]`, `threeSixty`, `augmentedReality`, `description`, `highlights[]`, `specifications[]`, `keyFeatures[]`, `dimensions[]`, `isBundle`, `bundleItems[]`, `subscription`, `installation`, `prop65`, `protectionPlanSkus[]`

**Variants and taxonomy** — `hasVariants`, `variantCount`, `variants[]`, `categories[]`, `breadcrumbs[]`, `department`, `badges[]`, `isSponsored`

**Store and fulfillment** — `storeId`, `storeName`, `storeCity`, `storeState`, `storePostalCode`, `storePhone`, `pickupAvailable`, `storeQuantity`, `pickupInStock`, `pickupLimited`, `curbsidePickup`, `pickupMaxQuantity`, `shipToStoreAvailable`, `deliveryAvailable`, `deliveryQuantity`, `deliveryEta`, `deliveryCharge`, `freeShipping`, `backordered`, `backorderDate`, `pickupOptions[]`, `deliveryOptions[]`, `aisle`, `bay`, `aisleBay`, `quantityLimit`, `returnPolicy`, `isBuyable`, `discontinued`

**Questions** — `questions[]` with the question, author, date, answers and photos

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | select | `search` | `search`, `category`, or `products` |
| `searchQueries` | list | – | Keywords for search mode, one per line |
| `categoryUrls` | list | – | Category `/b/...` URLs or browse codes (`N-...`) |
| `itemIds` | list | – | Item IDs or `/p/...` product URLs |
| `storeId` | string | `121` | Store number for price and shelf stock |
| `zipCode` | string | – | Finds the nearest store when `storeId` is empty; also used for delivery estimates |
| `sortBy` | select | `relevance` | `relevance`, `top_sellers`, `price_low_to_high`, `price_high_to_low`, `top_rated`, `newest` |
| `priceMin` / `priceMax` | number | – | USD price bounds |
| `brand` | string | – | Case-insensitive brand filter |
| `minRating` | number | – | Drop products rated below this value (0–5) |
| `onlyOnSale` | boolean | `false` | Keep real discounts, promotions, special buys and clearance |
| `onlyClearance` | boolean | `false` | Keep clearance products only |
| `onlyInStock` | boolean | `false` | Keep products available for pickup or delivery |
| `includeSponsored` | boolean | `false` | Include paid placements |
| `filterKeywords` | string | – | Comma-separated title words to keep |
| `maxItems` | integer | `100` | Overall product cap (1–5000) |
| `maxItemsPerQuery` | integer | `100` | Cap per query or category |
| `maxPages` | integer | `30` | Pages per source (1–30; 24 products per page) |
| `includeDetails` | boolean | `true` | Add specs, description, UPC/GTIN, media and variants |
| `includeReviews` | boolean | `false` | Add reviews and rating breakdown |
| `maxReviewsPerProduct` | integer | `20` | Review cap per product |
| `reviewsSort` | select | `most_recent` | `most_recent`, `most_helpful`, `oldest`, `highest_rating`, `lowest_rating`, `photos_first` |
| `includeQuestions` | boolean | `false` | Add customer questions and answers |
| `maxQuestionsPerProduct` | integer | `10` | Question cap per product |
| `includeStoreAvailability` | boolean | `true` | Add pickup/delivery stock fields |
| `includeAisleBay` | boolean | `false` | Add aisle and bay for in-store SKUs |

Legacy inputs from earlier versions (`searchQuery`, `startUrls`) continue to work and are mapped automatically. No proxy setup or API key is needed.

## Examples

### Search for drills under $300

```json
{
  "mode": "search",
  "searchQueries": ["cordless drill"],
  "storeId": "121",
  "priceMax": 300,
  "sortBy": "top_sellers",
  "maxItems": 50,
  "includeDetails": true,
  "includeStoreAvailability": true
}
```

### Browse a category and keep only clearance

```json
{
  "mode": "category",
  "categoryUrls": ["https://www.homedepot.com/b/Tools-Power-Tools-Drills/N-5yc1vZc27f"],
  "storeId": "121",
  "onlyClearance": true,
  "maxItems": 200,
  "maxPages": 10
}
```

### Look up specific item IDs with reviews and aisle locations

```json
{
  "mode": "products",
  "itemIds": ["320326787", "https://www.homedepot.com/p/Milwaukee-M18-FUEL-18V-Lithium-Ion-Brushless-Cordless-Hammer-Drill-and-Impact-Driver-Combo-Kit-2-Tool-with-2-Batteries-3697-22/320326787"],
  "storeId": "121",
  "includeDetails": true,
  "includeReviews": true,
  "maxReviewsPerProduct": 20,
  "includeStoreAvailability": true,
  "includeAisleBay": true
}
```

### Find the nearest store from a ZIP

```json
{
  "mode": "search",
  "searchQueries": ["pressure washer"],
  "storeId": "",
  "zipCode": "30339",
  "includeStoreAvailability": true,
  "maxItems": 25
}
```

## Use cases

- **Price monitoring** — compare store-local prices and promotions over time.
- **Restock and pickup tracking** — watch shelf pickup quantities at one store.
- **Assortment and catalog research** — pull a whole category with specs, brands and variants.
- **Comparison shopping** — collect models, UPCs, ratings and media for a shortlist.
- **Voice-of-customer research** — sample reviews and Q&A across competing products.

## FAQ

**Is this affiliated with Home Depot?**
No. This is an independent, third-party actor that reads publicly visible Home Depot product data. It is not operated by or affiliated with The Home Depot.

**Which store are the prices and stock from?**
They come from the `storeId` you enter (default 121, Cumberland, GA). If you leave `storeId` empty and supply `zipCode`, the nearest store is used. Change the store to see that store's price and shelf quantity.

**What is the difference between pickup and delivery stock?**
`storeQuantity` and `pickupInStock` are the shelf quantity available for pickup at the selected store. `deliveryQuantity`, `deliveryAvailable` and the delivery options describe the online/delivery pool. They are separate pools and are never added together.

**Why are some fields missing on a product?**
Home Depot does not expose every field for every product. Fields with no real value are omitted rather than filled with null, zero or a placeholder. Variants, specs, videos and aisle locations appear only when the product has them. Aisle and bay are added only for products with pickup stock at the selected store.

**How many products can one run return?**
`maxItems` is the overall cap and `maxItemsPerQuery` caps each query or category. Each page holds up to 24 products, and a category can be paged up to 30 times (720 products). For a larger category, narrow it by price or brand, or split it into smaller categories.

**Can I get every review for a product with thousands of reviews?**
No. Reviews are optional and capped by `maxReviewsPerProduct` (default 20). The actor reads review pages until it reaches that cap or the end of the review list. The total review count and rating breakdown still show the full picture, but only the requested number of review texts is saved.

**Do I need a proxy or an API key?**
No. Enter a store and the products you want; the actor handles access automatically.

**What happens if Home Depot refuses a request?**
The actor retries with a few fresh browser sessions. If every attempt is refused, the run ends with a status message naming the source and saves no placeholder rows; re-run later.

**Does this work for Home Depot Canada or Mexico?**
No. This actor covers the US storefront only. Canadian and Mexican stores are separate sites with different product IDs and stock.

**Why does the price differ from the first page I see on the website?**
The site personalises price and availability by store. Make sure the `storeId` in your input is the store you are comparing against.
