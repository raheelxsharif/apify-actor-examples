# TradeIndia B2B Product & Supplier Search Scraper

TradeIndia.com lists millions of products and manufacturer/exporter profiles across India's industrial, textile, chemical, food and engineering sectors. This actor runs a keyword search against TradeIndia and returns either the matching product listings or the matching supplier/company profiles as structured records, so you don't have to click through search result pages by hand.

## What this actor does

- Searches TradeIndia by free-text keyword in one of two modes: product listings or supplier/company listings
- Optionally narrows results to a single Indian city
- Pulls price/price-range, minimum order quantity and product image for each product result
- Pulls company name, city, product categories and phone number for each supplier result
- Empty fields are omitted from records — a record never contains a null or empty value

## Output per product (mode = `searchProducts`)

- `productId` — identifier parsed from the product's TradeIndia URL
- `title` — product name as listed
- `price` — price or price range text as shown by the supplier (usually INR)
- `minOrder` — minimum order quantity/unit text
- `supplier` — selling company's name
- `supplierCity` — city where the supplier is based
- `imageUrl` — product photo URL
- `productUrl` — link to the TradeIndia product page
- `recordType` — always `product`
- `scrapedAt` — UTC timestamp when the record was collected

## Output per supplier (mode = `searchSuppliers`)

- `supplierId` — identifier parsed from the company's TradeIndia URL
- `supplier` — company name
- `supplierCity` — city where the company is based
- `products` — product categories/keywords the company lists
- `phone` — contact phone number, when published on the listing
- `supplierUrl` — link to the TradeIndia company profile
- `recordType` — always `supplier`
- `scrapedAt` — UTC timestamp when the record was collected

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `searchProducts` | `searchProducts` to return product listings, `searchSuppliers` to return company/supplier listings |
| `keyword` | string | `rubber gaskets` | Product or supplier keyword to search for |
| `city` | string | – | Restrict results to a specific Indian city (e.g. `Mumbai`, `Delhi`, `Chennai`); matched against the supplier's listed city |
| `maxItems` | integer | `20` | Maximum number of records to return (1–500) |

### Example: sourcing rubber gaskets

```json
{
  "mode": "searchProducts",
  "keyword": "rubber gaskets",
  "maxItems": 50
}
```

### Example: suppliers of cotton fabric in Mumbai

```json
{
  "mode": "searchSuppliers",
  "keyword": "cotton fabric",
  "city": "Mumbai",
  "maxItems": 30
}
```

### Example: broad export lead list for LED lights

```json
{
  "mode": "searchSuppliers",
  "keyword": "LED lights",
  "maxItems": 200
}
```

## Use cases

- **Import/export sourcing** — shortlist Indian manufacturers and exporters for a product category before reaching out
- **Price benchmarking** — compare listed prices and MOQs for the same product across multiple sellers
- **B2B lead generation** — build contact lists of suppliers filtered to a specific city or region
- **Market research** — gauge how many sellers and what price spread exists for a given keyword on India's largest B2B marketplace
- **Procurement shortlisting** — narrow a long supplier list down to companies based in a target city

## FAQ

**What does this actor search?** It runs the same keyword search box available on TradeIndia.com, in either product or supplier mode, and parses the result pages into structured records.

**Is a TradeIndia account or login required?** No. Only the publicly visible search results are read; no login, cookies or API key are needed.

**Why do some records have no `price` or `phone`?** TradeIndia sellers choose what to publish on their listing. If a seller didn't list a price, MOQ or phone number, that field is left out of the record rather than returned empty.

**Can I search for suppliers in a specific city without a product keyword?** A `keyword` is required, but you can pair it with the `city` filter to narrow results — for example searching "cotton fabric" with `city` set to "Mumbai" only returns suppliers based there.

**How many results can I get per run?** Up to the `maxItems` value you set, capped at 500. The actor stops early if TradeIndia's search runs out of further result pages for the keyword.

**Is this affiliated with TradeIndia?** No. This is an independent, third-party actor that reads publicly available TradeIndia search results; it is not operated by or affiliated with TradeIndia.com.

**Why do city filter results sometimes come back empty?** The `city` filter matches against the city text shown on each listing, so uncommon spellings or suppliers that didn't list a city won't match — try a broader keyword or remove the city filter.
