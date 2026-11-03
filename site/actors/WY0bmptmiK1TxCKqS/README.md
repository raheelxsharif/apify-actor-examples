# Global B2B Supplier Finder by Product & Country

Kompass lists tens of millions of manufacturers, distributors, and service companies across dozens of countries. This actor searches that directory by company name, product keyword, or country, or pulls full details for profiles you already have the URL for, and returns structured company records for sourcing and lead work.

## What this actor does

- **Four search modes**: free-text company search, product/service keyword search, country browse, and direct profile lookup by URL
- **Filters by country** using a 32-country dropdown covering North America, Europe, Asia, and more
- **Returns firmographic detail** — employee count range, revenue range, certifications, and products/services offered
- **Captures contact points** — primary contact name, phone number, and company website when listed
- **Empty fields are omitted** from each record instead of being returned as null

## Output per company

- `companyId` — Kompass's internal company identifier
- `companyName` — full registered company name
- `country` — company's country
- `city` — city or region
- `employees` — employee count range
- `revenue` — revenue range
- `products` — products and services offered
- `certifications` — quality certifications (e.g. ISO)
- `websiteUrl` — company's own website
- `companyUrl` — Kompass profile URL
- `contactName` — primary contact person name
- `phone` — company phone number
- `scrapedAt` — UTC timestamp of when the record was scraped

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string (enum, required) | `searchCompanies` | `searchCompanies`, `searchByProduct`, `searchByCountry`, or `companyDetails` |
| `query` | string | `machine tools` | Company name or keyword (modes `searchCompanies` / `searchByProduct`) |
| `country` | string (enum) | `US` | 2-letter country code filter, or blank for all countries |
| `companyUrls` | array | `[]` | Kompass company profile URLs to fetch (mode `companyDetails`) |
| `maxItems` | integer | `20` | Maximum number of company records to return (1–500) |

### Example: search by product keyword in one country

```json
{
  "mode": "searchByProduct",
  "query": "industrial valves",
  "country": "DE",
  "maxItems": 30
}
```

### Example: browse all companies listed for a country

```json
{
  "mode": "searchByCountry",
  "country": "IN",
  "maxItems": 50
}
```

### Example: fetch specific company profiles

```json
{
  "mode": "companyDetails",
  "companyUrls": [
    "https://www.kompass.com/c/example-company/de123456/"
  ]
}
```

## Use cases

- **B2B lead generation** — build a list of companies in a target country and product category
- **Supplier sourcing** — shortlist manufacturers with the certifications a buyer requires
- **Market mapping** — see how many and which companies serve a given product category per country
- **Export/import partner search** — find counterparts in a specific country for a trade relationship
- **Company verification** — pull employee count, revenue range, and contact details for due diligence

## FAQ

**Is this affiliated with Kompass?**
No. This is an independent, third-party actor that reads publicly visible Kompass pages; it isn't operated by or affiliated with Kompass.

**Do I need a Kompass account?**
No. Public search and company profile pages don't require a login.

**What's the difference between `searchCompanies` and `searchByProduct`?**
`searchCompanies` matches your query against company names and general listing text. `searchByProduct` matches it specifically against the products and services companies list, which surfaces different results for the same keyword.

**Why is `revenue` or `certifications` sometimes missing?**
Not every Kompass profile publishes every field. Fields with no value on the source page are omitted from the record rather than returned empty.

**Can I search without filtering by country?**
Yes, leave `country` blank to search across all countries Kompass covers.

**How current is the data?**
Each run reads Kompass's live pages, so company details reflect what's published at run time.

**What if I already have a list of company URLs?**
Use `mode: "companyDetails"` with `companyUrls` to fetch full details for exactly those profiles instead of running a search.
