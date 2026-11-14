# Flippa Business & Website Listing Search Scraper

Flippa is the largest marketplace for buying and selling websites, ecommerce stores, SaaS products, mobile apps, domains and other online businesses. This actor queries Flippa's search catalog — either by pasting a Flippa search URL or by setting structured filters — and returns every matching listing as a clean record with financials, traffic, verification and seller data.

## What this actor does

- Searches Flippa listings by pasting any `flippa.com/search` URL, or by setting structured filter fields (property type, status, price, revenue, profit, verification, keyword)
- Covers every Flippa property type: websites, ecommerce stores, SaaS, iOS/Android apps, domains, newsletters, digital products, content sites and services
- Returns financial metrics (price, revenue, profit, multiples), traffic metrics, and Flippa's own revenue/traffic verification flags for each listing
- Sorts results by newest, price, revenue or profit, and paginates automatically up to the requested item count
- Empty fields are omitted from records — a field only appears when Flippa actually published that data for the listing

## Output per listing

- `id` — Flippa listing ID
- `title` — listing title
- `url` — full Flippa listing URL
- `property_name` — property/brand name
- `property_type` — asset type as returned by Flippa (e.g. `ecommerce_store`, `shopify`, `beehiiv`)
- `industry` — industry/niche
- `hostname` — primary hostname of the property
- `external_url` — the property's own external URL
- `summary` — seller's short description
- `price` — current/display price in USD
- `buy_it_now_price` — buy-it-now price, for auctions
- `sale_method` — `auction` or `classified`
- `status` — listing status
- `bid_count` — number of bids, for auctions
- `reserve_met` — whether an auction's reserve price has been met
- `revenue_per_month`, `profit_per_month` — latest monthly revenue/profit in USD
- `average_revenue`, `average_profit` — average monthly revenue/profit
- `revenue_multiple`, `profit_multiple` — price divided by annualized revenue/profit
- `page_views_per_month`, `uniques_per_month` — monthly traffic metrics
- `app_downloads_per_month` — monthly app downloads, for app listings
- `has_verified_revenue`, `has_verified_traffic` — Flippa verification flags
- `established_at`, `age_years` — when the property was established and its age
- `starts_at`, `ends_at` — listing start/end time
- `seller_location` — seller's country
- `super_seller` — `true` when the seller holds Flippa's Super Seller status
- `turnkey_listing` — `true` when tagged as a turnkey business
- `confidential` — `true` when the listing is marked confidential
- `revenue_sources` — revenue source tags
- `image_url` — listing thumbnail URL
- `scraped_at` — UTC timestamp when the record was collected

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `searchUrl` | string | – | A full `flippa.com/search` URL; its filters are mapped directly to Flippa's search. Leave empty to use the structured filter fields below instead |
| `maxItems` | integer | `50` | Maximum listings to return (1–10,000) |
| `propertyType` | array | – | Listing categories to include (used only when `searchUrl` is empty): `website`, `ecommerce_store`, `saas`, `ios_app`, `android_app`, `domain`, `newsletter`, `digital_product`, `content`, `service`. Leave blank for all types |
| `status` | string | `open` | `open`, `under_offer`, or `closed` |
| `saleMethod` | string | – | `auction` or `classified` |
| `priceMin` / `priceMax` | integer | – | Asking price range in USD |
| `revenueMin` | integer | – | Minimum monthly revenue in USD |
| `profitMin` | integer | – | Minimum monthly profit in USD |
| `verifiedRevenue` | boolean | `false` | Only include listings with Flippa-verified revenue |
| `verifiedTraffic` | boolean | `false` | Only include listings with Flippa-verified traffic |
| `searchQuery` | string | – | Keyword search, e.g. `shopify`, `saas`, `blog` |
| `sort` | string | `-created_at` | `-created_at`, `created_at`, `-price`, `price`, `-revenue_per_month`, `-profit_per_month`, or `most_relevant` |

### Example: open ecommerce stores from a pasted URL

```json
{
  "searchUrl": "https://flippa.com/search?filter%5Bproperty_type%5D%5B%5D=ecommerce_store&filter%5Bstatus%5D=open",
  "maxItems": 100
}
```

### Example: verified SaaS businesses over $10k revenue

```json
{
  "propertyType": ["saas"],
  "revenueMin": 10000,
  "verifiedRevenue": true,
  "sort": "-profit_per_month",
  "maxItems": 50
}
```

### Example: keyword search across all types

```json
{
  "searchQuery": "shopify",
  "priceMin": 5000,
  "priceMax": 100000,
  "maxItems": 75
}
```

## Use cases

- **Deal sourcing** — scan open listings filtered by profit multiple and verified revenue to find under-priced acquisition targets
- **Niche monitoring** — track the newsletter, content or digital-product categories for new deal flow
- **SaaS acquisition research** — filter SaaS listings by monthly revenue and age to build a shortlist
- **App portfolio scouting** — scan iOS and Android app listings with verified download counts
- **Market benchmarking** — compare price-to-revenue and price-to-profit multiples across industries and property types

## FAQ

**Do I need a Flippa account or API key?** No — this actor reads Flippa's public search catalog; no login or credentials are required.

**Can I get sold listings?** No. Flippa's public catalog does not expose sold listings, so only `open`, `under_offer` and `closed` (unsold, ended) listings are available — and in practice `under_offer`/`closed` currently return no records at all, only `open` does.

**How many listings can one run return?** Up to `maxItems`, capped at 10,000. For very broad categories, narrow with price or revenue filters to stay under that cap.

**Why does `price` sometimes exceed the `priceMax` I set?** The price filter is applied against the listing's asking/buy-it-now price at query time. For auctions, the current bid shown in `price` can rise above that as bidding progresses, while `buy_it_now_price` keeps the original asking price. Filter on `saleMethod: "classified"` if you need the returned price to always stay inside your range.

**Why does `property_type` not exactly match the category I filtered by?** Flippa filters on a high-level category (e.g. `newsletter`) but the record's `property_type` reports the more specific platform Flippa detected (e.g. `beehiiv`). The filter still worked correctly — group by the filter you passed rather than the returned value if you need the broad category.

**Are `digital_product` and `content` property types useful?** They're valid filter values, but at any given time Flippa may have zero active listings in those categories; the actor returns an empty dataset rather than an error when that happens.

**How fresh is the data?** Every run queries Flippa live, so results reflect the marketplace at the moment the actor runs.

**Is this affiliated with Flippa?** No. This is an independent, third-party actor that reads Flippa's public listing data; it is not operated by or affiliated with Flippa.com.
