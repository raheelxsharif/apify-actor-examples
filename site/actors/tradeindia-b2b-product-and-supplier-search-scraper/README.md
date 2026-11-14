# TradeIndia B2B Product & Supplier Scraper

Turn TradeIndia.com into a structured sourcing database. Search India's largest B2B marketplace by keyword, or paste product and company links, and get back prices, minimum order quantities, specifications, supplier locations, business types, GST numbers, owners and published contact details.

The actor talks to the same public JSON services that power TradeIndia's own search and profile pages. There is no browser, no login, no cookie and no API key.

## What you can collect

| Mode | What it does | Best for |
|---|---|---|
| `searchProducts` | Keyword search across product listings, with city and supplier-type filters and pagination | Price benchmarking, category research, product discovery |
| `searchSuppliers` | Keyword search that aggregates every listing into one record per company | Manufacturer/exporter lead lists, distributor discovery |
| `productDetails` | Full product page by URL or numeric product ID | Specifications, MOQ, stock, GST price, company profile |
| `supplierDetails` | Full company profile by URL or numeric company ID | Verified supplier due diligence, contact and turnover data |

Search modes accept several queries and several cities in one run. The actor stops when it reaches `maxItems` or when TradeIndia has no more result pages — there is no wall-clock cut-off.

## Product record fields

| Field | Description |
|---|---|
| `productId` | TradeIndia product ID |
| `title`, `longTitle` | Product name and the longer SEO title |
| `description` | Short product description |
| `category`, `productType` | Category and specification-derived product type |
| `brandName` | Brand, when published |
| `price`, `priceValue`, `currency` | Price as shown plus a numeric value and ISO currency |
| `gstPrice`, `gstRate`, `mrp` | GST-inclusive price, GST rate and MRP (details modes) |
| `minOrder`, `minOrderQuantity`, `unit` | Minimum order text, numeric quantity and unit |
| `stockQuantity`, `inStock`, `shopOnline` | Stock and online-purchase signals |
| `specifications` | Key/value product specifications |
| `tradeInformation` | Delivery time, supply ability, payment terms, packaging, markets, certifications |
| `priceAndQuantity`, `shopDetails` | Raw price/MOQ and Ti Shopping detail sections |
| `imageUrl`, `imageUrls` | Main image and additional images |
| `productUrl` | Absolute product page URL |
| `keywords` | Supplier keywords/business categories (search modes) |
| `supplier`, `supplierId`, `supplierUrl` | Selling company |
| `supplierCity`, `supplierState`, `supplierCountry`, `supplierAddress` | Location |
| `supplierBusinessType` | Manufacturer / supplier / exporter / trader / distributor |
| `supplierRating`, `buyerFeedbackRating` | Buyer feedback scores when published (search modes) |
| `supplierGst`, `supplierOwner`, `supplierDesignation` | Company registration and contact person |
| `supplierEmployees`, `supplierTurnover`, `supplierEstablishmentYear` | Company size and age |
| `supplierMemberSince`, `supplierTrustStamp`, `supplierVerified` | Tenure and verification badges |
| `supplierLogo`, `supplierDescription`, `supplierMapUrl` | Company branding and profile text |
| `supplierOtherProducts` | More products from the same company (details/enrichment) |
| `phone`, `email`, `websiteUrl` | Published contact details |
| `recordType`, `scrapedAt` | `product` and collection timestamp |

## Supplier record fields

Supplier search records carry every company field available from the listing. Supplier detail records add the full profile surface:

| Field | Description |
|---|---|
| `supplierId`, `supplier`, `supplierUrl` | Company identity |
| `supplierCity`, `supplierState`, `supplierCountry`, `supplierAddress` | Location |
| `businessType`, `businessTypes` | Trading role(s) |
| `memberSinceYears` | Years on TradeIndia |
| `ownerName`, `designation` | Proprietor/contact person |
| `gstNumber` | GST registration |
| `employeesCount`, `annualTurnover`, `establishmentYear` | Company scale |
| `workingDays`, `certifications`, `responseRate` | Operations |
| `rating`, `buyerFeedbackRating` | Buyer feedback |
| `companyDescription`, `companyLogo`, `websiteUrl` | Profile |
| `latitude`, `longitude`, `mapUrl` | Location |
| `categories`, `productNames`, `productCount` | Product portfolio |
| `productServices` | Products grouped by category |
| `faqs` | Company FAQ answers from the profile |
| `phone`, `email` | Published contact details |
| `trustStamp`, `verified`, `hasPrice`, `shopOnline` | Lead-quality signals |
| `recordType`, `scrapedAt` | `supplier` and collection timestamp |

Empty fields are omitted at every depth: a record never contains `null`, `""`, `[]` or `{}`.

## Inputs

| Field | Type | Default | Notes |
|---|---|---|---|
| `mode` | select | `searchProducts` | One of the four modes above |
| `searchQueries` | string list | `["rubber gaskets"]` | One or more keywords; every query is combined with every city |
| `cities` | string list | `[]` | Indian city names; TradeIndia applies this filter server-side |
| `startUrls` | string list | `[]` | Product/company URLs or numeric IDs for the details modes |
| `businessTypes` | multi-select | `[]` | Manufacturer, supplier, exporter, trader, wholesaler/distributor |
| `withPrice` | boolean | `false` | Drop listings without a price |
| `trustStamp` | boolean | `false` | Keep Ti Trusted suppliers |
| `buyerFeedbackOnly` | boolean | `false` | Keep suppliers with buyer feedback |
| `shopOnline` | boolean | `false` | Keep products available for online purchase |
| `includeDetails` | boolean | `false` | Search modes: fetch full detail pages per record |
| `includeContact` | boolean | `true` | Add published phone/email where available |
| `maxItems` | integer | `20` | 1–5000 records |
| `maxProductsPerSupplier` | integer | `12` | 1–120 products on a supplierDetails record |

### Examples

**Price benchmark for rubber gaskets in Mumbai**

```json
{
  "mode": "searchProducts",
  "searchQueries": ["rubber gaskets"],
  "cities": ["Mumbai"],
  "withPrice": true,
  "maxItems": 100
}
```

**Manufacturer lead list for LED lights**

```json
{
  "mode": "searchSuppliers",
  "searchQueries": ["LED lights", "LED bulbs"],
  "businessTypes": ["manufacturer", "exporter"],
  "includeDetails": true,
  "maxItems": 50
}
```

**Full product page**

```json
{
  "mode": "productDetails",
  "startUrls": ["https://www.tradeindia.com/products/rubber-cut-gasket-c10220130.html"],
  "maxItems": 1
}
```

**Full company profile by numeric ID**

```json
{
  "mode": "supplierDetails",
  "startUrls": ["357545"],
  "maxProductsPerSupplier": 24,
  "maxItems": 1
}
```

## How filters behave

`cities` is applied by TradeIndia's search service. The other search filters (`businessTypes`, `withPrice`, `trustStamp`, `buyerFeedbackOnly`, `shopOnline`) are applied by the actor after each page is fetched, because the public keyword endpoint ignores those flags. The actor keeps paging until it fills `maxItems` or the result set is exhausted; the status message reports how many listings were scanned.

`searchSuppliers` builds one record per company from the product result pages. With `includeDetails` on, each company profile is then fetched to fill GST, owner, turnover, employee count, address and product portfolio. `searchProducts` with `includeDetails` fetches the matching product page for every product, adding specifications, stock and company details.

## Data source and limitations

- TradeIndia publishes up to 50 listing rows per request; the actor paginates through every available page for each query/city pair.
- Supplier contact details (phone/email), GST, owner and company descriptions are included only when TradeIndia publishes them on the public page. Some sellers hide their number or email; those fields are omitted rather than guessed.
- Supplier-search records are aggregated from product listings, so a company appears once even if it sells many matching products. Turn `includeDetails` on for the full company profile.
- TradeIndia changes its catalogue and page layout over time. If a run suddenly returns no rows, check the run log and the status message before assuming the query has no results.
- This is an independent third-party tool. It is not affiliated with, endorsed by, or operated by TradeIndia.com.

## Legacy input compatibility

Older saved tasks used `keyword`, `city`, `maxResults` and the same two search mode values. The actor still reads those keys from the raw input and maps them onto `searchQueries`, `cities` and `maxItems`; they are intentionally not shown in the input form. Every published output field from the previous version is still present.

## FAQ

**Do I need a TradeIndia account?** No. Only public search results and public company/product pages are read.

**Why are some prices missing?** Sellers choose whether to publish a price. The `withPrice` filter removes those records, or leave it off and handle the gaps yourself.

**Can I search several keywords at once?** Yes. Put each keyword in `searchQueries`; results are deduplicated by product or company ID.

**Can I get every product a supplier sells?** `supplierDetails` returns the portfolio TradeIndia renders on the profile (up to `maxProductsPerSupplier`) plus the category groupings. The full catalogue may be larger than the public profile pages expose.

**Is a proxy required?** No. The actor talks directly to TradeIndia's public JSON services.
