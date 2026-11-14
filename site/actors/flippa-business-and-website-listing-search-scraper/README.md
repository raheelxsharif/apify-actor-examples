# Flippa Business & Website Listing Search Scraper

Flippa is a marketplace for buying and selling websites, ecommerce stores, SaaS products, apps, domains, newsletters and other online businesses. This actor reads Flippa's public search results and listing pages — with no account, login or cookie — and returns one structured record per listing.

## Two ways to collect data

| Mode | What it does | Inputs |
|---|---|---|
| `search` (default) | Pages through Flippa search results and returns one record per listing card. | Structured filters or a pasted `flippa.com/search` URL. |
| `listingDetails` | Fetches the full public detail page for exact listings and adds description, margin, auction status, seller and verification data. | One or more Flippa listing URLs or numeric listing IDs. |

Both modes deduplicate by Flippa listing ID, stop at `maxItems`, and omit fields Flippa does not publish for a listing.

## Search filters

Every filter below is applied by Flippa's own search endpoint, so the returned rows honour it. Results are paginated automatically at up to 100 rows per page.

| Input | Values / notes |
|---|---|
| `categories` | Ecommerce, Content, SaaS, YouTube Channels, Amazon KDP, Amazon FBA, iOS Apps, Android Apps, Digital Agencies, Services, AI Apps & Tools, Marketplaces, Social Media Accounts, Newsletters, Plugins & Extensions, Crypto & Blockchain, Projects & Concepts, Domains, Other |
| `statuses` | `open` or `won` (sold). Defaults to `open`. |
| `saleMethods` | `auction`, `classified` (fixed asking price) |
| `managedBy` | `owner`, `flippa_broker`, `partner_broker` |
| `verticals` | Automotive, Business, Design & Style, Education, Electronics, Entertainment, Lifestyle, Food & Drink, General Knowledge, Health & Beauty, Hobbies & Games, Home & Garden, Internet, Sports & Outdoor, Travel |
| `keyword` | Free-text search, e.g. `shopify`, `saas`, `newsletter` |
| `sortBy` | Most Relevant, Most Active, Lowest Price, Highest Price, Most Recent, Ending Soonest, Most Profitable |
| `priceMin` / `priceMax` | USD price range |
| `revenueMin` / `revenueMax` | USD monthly revenue range |
| `profitMin` / `profitMax` | USD monthly profit range |
| `ageMin` / `ageMax` | Business age in years |
| `multipleMin` / `multipleMax` | Profit multiple range, e.g. `2.5` means 2.5× annual profit |
| `revenueMultipleMin` / `revenueMultipleMax` | Revenue multiple range, e.g. `2.5` means 2.5× annual revenue |
| `uniquesMin` / `uniquesMax` | Monthly unique visitors |
| `pageViewsMin` / `pageViewsMax` | Monthly page views |
| `downloadsMin` / `downloadsMax` | Monthly app downloads |
| `verifiedRevenue` / `verifiedTraffic` | Keep only Flippa-verified listings |
| `searchUrl` | Paste any Flippa search or category URL; its filters are reused. |

Saved tasks that still send the older keys `propertyType`, `status`, `saleMethod`, `searchQuery`, `sort`, `startUrl` and `maxResults` keep working: the actor maps them in code to the equivalent new filters, even when the input form has filled the newer field with its default value.

## Output

Each record is a flat JSON object. Fields appear only when Flippa publishes that value, so no `null`, empty string or empty list is ever written.

**Identity and classification:** `id`, `title`, `url`, `property_name`, `property_type`, `category`, `industry`, `summary`, `description`, `hostname`, `external_url`.

**Price and sale state:** `price`, `price_type` (`asking_price`, `starting_bid`, `highest_bid`, `sold_price`), `price_on_request`, `currency`, `buy_it_now_price`, `sale_method`, `status`, `bid_count`, `reserve_met`, `auction_ends_in_days`.

**Financials:** `revenue_per_month`, `profit_per_month`, `profit_margin`, `average_revenue`, `average_profit`, `revenue_multiple`, `profit_multiple`.

**Traffic and verification:** `page_views_per_month`, `uniques_per_month`, `app_downloads_per_month`, `has_verified_revenue`, `has_verified_traffic`, `data_verified`, `verified_listing`.

**Seller and metadata:** `age_years`, `established_at`, `starts_at`, `ends_at`, `seller_location`, `seller_name`, `super_seller`, `turnkey_listing`, `confidential`, `editors_choice`, `sponsored`, `managed_by_flippa`, `badges`, `revenue_sources`, `comments_count`, `views_count`, `watchers_count`, `image_url`, `scraped_at`.

## Examples

Open ecommerce stores:

```json
{
  "mode": "search",
  "categories": ["ecommerce"],
  "statuses": ["open"],
  "maxItems": 25
}
```

Verified SaaS listings above $10k monthly revenue, most profitable first:

```json
{
  "mode": "search",
  "categories": ["saas"],
  "revenueMin": 10000,
  "verifiedRevenue": true,
  "sortBy": "most_profitable",
  "maxItems": 50
}
```

Recently sold businesses:

```json
{
  "mode": "search",
  "statuses": ["won"],
  "maxItems": 25
}
```

Full details for exact listings (URLs or numeric IDs):

```json
{
  "mode": "listingDetails",
  "listingUrls": [
    "https://flippa.com/11674627-profitable-canadian-rooftop-tent-brand",
    "12061800"
  ],
  "maxItems": 2
}
```

## Pagination and limits

- Search pages hold up to 100 rows. The actor follows pages until `maxItems` is reached, Flippa stops returning rows, or the 10,000-result search window is exhausted.
- Flippa reports some broad categories as "10,000+" matches. Narrow by price, revenue, age, industry or sale method to stay inside the window.
- `listingDetails` accepts up to 1,000 URLs or IDs per run.

## FAQ

**Do I need a Flippa account, API key, cookie or proxy?** No. The actor reads public search and detail pages directly.

**Why is `price_type` sometimes `starting_bid` instead of `asking_price`?** Flippa shows auctions with a starting bid and classified listings with an asking price. The field tells you which price is in `price`.

**Why are some fields missing?** Flippa hides financials for confidential listings, hides traffic until sign-up for some sellers, and does not publish every metric for every listing. Missing means "not public", never "zero".

**Can I get sold listings?** Yes — set `statuses` to `["won"]`. Sold listings may show "Contact Flippa for Price" instead of a public price; those rows set `price_on_request` and omit `price`.

**Why does the card price differ slightly from the detail page?** Search cards use Flippa's rounded USD display (for example `$102K`); detail pages return the exact public value. Use `listingDetails` when exact numbers matter.

**Is this affiliated with Flippa?** No. This is an independent third-party actor that reads publicly available Flippa listings.

## Input reference

| Input | Type | Default |
|---|---|---|
| `mode` | select | `search` |
| `searchUrl` | string | – |
| `listingUrls` | string list | – |
| `maxItems` | integer | `50` |
| `categories` | multi-select | – |
| `statuses` | multi-select | `["open"]` |
| `saleMethods` | multi-select | – |
| `managedBy` | multi-select | – |
| `verticals` | multi-select | – |
| `keyword` | string | – |
| `sortBy` | select | `most_relevant` |
| `priceMin` | integer | – |
| `priceMax` | integer | – |
| `revenueMin` | integer | – |
| `revenueMax` | integer | – |
| `profitMin` | integer | – |
| `profitMax` | integer | – |
| `ageMin` | number | – |
| `ageMax` | number | – |
| `multipleMin` | number | – |
| `multipleMax` | number | – |
| `revenueMultipleMin` | number | – |
| `revenueMultipleMax` | number | – |
| `uniquesMin` | integer | – |
| `uniquesMax` | integer | – |
| `pageViewsMin` | integer | – |
| `pageViewsMax` | integer | – |
| `downloadsMin` | integer | – |
| `downloadsMax` | integer | – |
| `verifiedRevenue` | boolean | `false` |
| `verifiedTraffic` | boolean | `false` |
