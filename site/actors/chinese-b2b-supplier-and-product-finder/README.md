# Made-in-China B2B Sourcing & Supplier Finder

Made-in-China.com is where Chinese factories and trading companies publish products, prices, certifications, audit badges, and company profiles. This actor turns those public pages into structured sourcing data: six modes, multi-keyword search, category browsing, product and supplier URLs, deep detail enrichment, and a computed lead score. No account, cookies, or proxy setup is needed.

## What this actor does

- **Six modes:** `searchProducts`, `searchSuppliers`, `categoryBrowse`, `productDetails`, `supplierDetails`, `supplierProducts`
- **Multi-input runs:** several keywords, categories, product URLs/IDs, or supplier storefronts per run
- **Faceted product search:** Chinese province, business type, R&D/production capability, certification, price range, minimum order quantity, sample availability, and supplier trust badges
- **Detail enrichment:** description, specifications, price tiers, images, video, categories, FAQ, HS code, origin, packaging, and supplier firmographics
- **Supplier enrichment:** membership tier, years on platform, audit agency/report, plant area, employees, capital, trade terms, production capacity, markets, product groups, and contact-page details
- **Numeric commerce fields:** `priceMin`, `priceMax`, `priceCurrency`, `priceTiers`, `minOrderQuantity`, and `minOrderUnit`, beside the legacy text fields
- **Lead score:** 0–100, computed from public trust, experience, trade, contact, and facility signals
- **Pagination and dedupe:** walks listing pages until `maxItems` or source exhaustion, dedupes canonical URLs, and persists crawl state
- **Omit-empty output:** a field appears only when the source published it; no nulls, blanks, or sentinel strings

## Modes

| Mode | What it does | Main inputs |
|---|---|---|
| `searchProducts` | Product keyword search with all filters | `searchQueries`, filters |
| `searchSuppliers` | Supplier/company keyword search | `searchQueries`, filters |
| `categoryBrowse` | Walk a Made-in-China catalog path | `categoryPaths` |
| `productDetails` | Scrape specific product URLs or IDs | `productUrls` |
| `supplierDetails` | Scrape supplier storefronts and profiles | `supplierUrls` |
| `supplierProducts` | Walk a supplier's own product catalog | `supplierUrls` |

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | select | `searchProducts` | One of the six modes above. |
| `searchQueries` | string list | `["industrial pump"]` | Product or supplier keywords (`searchProducts`, `searchSuppliers`). |
| `categoryPaths` | string list | – | Catalog paths or full catalog URLs (`categoryBrowse`), e.g. `Agriculture-Food-Catalog/Beverage`. |
| `productUrls` | string list | – | Product URLs or bare product IDs (`productDetails`). |
| `supplierUrls` | string list | – | Storefront URLs or subdomains such as `allslorm` (`supplierDetails`, `supplierProducts`). |
| `province` | select | – | Chinese province/municipality/region filter. |
| `businessType` | select | – | Manufacturer/Factory, Trading Company, Group Corporation, or Other. |
| `goldSupplierOnly` | boolean | `false` | Gold/Diamond membership or higher. |
| `auditedSupplierOnly` | boolean | `false` | Supplier has an independent audit report. |
| `leadingFactoryOnly` | boolean | `false` | Supplier holds the Industry-leading Audited Factory badge. |
| `securedTradingOnly` | boolean | `false` | Listing offers Secured Trading. |
| `rdCapacity` | select | – | OEM, ODM, Own Brand, or Others. |
| `certification` | select | – | CE, ISO, RoHS, CCC, or ISO9001. |
| `maxMinOrder` | integer | – | Only products with an MOQ at or below this number. |
| `priceMin` | number | – | Lower USD price bound; set together with `priceMax`. |
| `priceMax` | number | – | Upper USD price bound; set together with `priceMin`. |
| `sampleAvailableOnly` | boolean | `false` | Supplier offers a paid sample order. |
| `maxItems` | integer | `50` | Maximum records across all inputs (1–10,000). |
| `maxPagesPerQuery` | integer | `20` | Listing pages per keyword/category/supplier (1–400). |
| `includeProductDetails` | boolean | `true` | Fetch product detail pages for deeper fields. |
| `includeSupplierProfile` | boolean | `true` | Fetch supplier storefront/about pages for firmographics. |
| `includeContactDetails` | boolean | `false` | Fetch the public supplier contact page where available. |

Legacy saved tasks that still send `keyword` or `categoryPath` keep working: the code maps them to `searchQueries` and `categoryPaths` when the new fields are absent.

### Example: search products by keyword

```json
{
  "mode": "searchProducts",
  "searchQueries": ["industrial pump", "submersible pump"],
  "province": "Guangdong",
  "auditedSupplierOnly": true,
  "maxItems": 50,
  "maxPagesPerQuery": 2,
  "includeProductDetails": true
}
```

### Example: search suppliers

```json
{
  "mode": "searchSuppliers",
  "searchQueries": ["office chair"],
  "includeSupplierProfile": true,
  "includeContactDetails": true,
  "maxItems": 20
}
```

### Example: scrape product or supplier URLs

```json
{
  "mode": "productDetails",
  "productUrls": [
    "https://allslorm.en.made-in-china.com/product/bYDpSVyclXkh/China-Qmf-G-Spiral-Centrifugal-Pump-for-Industrial-Drainage.html"
  ],
  "maxItems": 10
}
```

```json
{
  "mode": "supplierProducts",
  "supplierUrls": ["allslorm"],
  "includeProductDetails": true,
  "maxItems": 30
}
```

## Product output

Identity and commerce:

`productId`, `title`, `productUrl`, `productSku`, `productMpn`, `modelNumber`, `price` (legacy text), `priceMin`, `priceMax`, `priceCurrency`, `priceValidUntil`, `priceTiers`, `minOrder` (legacy text), `minOrderQuantity`, `minOrderUnit`, `samplePrice`, `samplePriceCurrency`, `availability`, `description`.

Content:

`images`, `imageUrl`, `hasVideo`, `videoUrl`, `videoThumbnail`, `specifications`, `categories`, `faq`, `hsCode`, `origin`, `trademark`, `transportPackage`, `productionCapacity`, `customization`.

Supplier and provenance:

`supplierId`, `supplier`, `supplierUrl`, `supplierLogo`, `supplierDescription`, `supplierProvince`, `supplierCity`, `supplierCountry`, `supplierAddress`, `supplierLevel`, `goldSupplier`, `membership`, `memberSince`, `yearsOnPlatform`, `auditedSupplier`, `leadingFactory`, `securedTrading`, `auditAgency`, `auditReportId`, `foundingDate`, `employees`, `plantArea`, `registeredCapital`, `supplierRating`, `reviewHighlight`, `businessType`, `certifications`, `mainMarkets`, `paymentTerms`, `incoterms`, `leadTime`, `nearestPort`, `exportYear`, `productionLines`, `productionMachines`, `qaQcInspectors`, `rdEngineers`, `oemService`, `odmService`, `ownBrand`, `customizationOptions`, `repeatBuyersRate`, `supplyChainPartners`, `productGroups`, `leadScore`, `searchKeyword`, `page`, `position`, `isSponsored`, `recordType: "product"`, `scrapedAt`.

Price tiers are objects with `quantityRange`, `quantityMin`, `quantityMax`, `unit`, `price`, and `currency`. `priceMin` is the lowest tier price (usually the highest quantity), not necessarily the headline price.

## Supplier output

Identity and legacy fields:

`supplierId`, `supplier`, `supplierProvince`, `businessType`, `products` (legacy text), `mainProducts`, `certifications` (legacy text), `managementCertifications`, `productCertifications`, `rdCapacity`, `goldSupplier`, `auditedSupplier`, `supplierUrl`.

Firmographics and trade:

`supplierDescription`, `introduction`, `foundingDate`, `employees`, `plantArea`, `registeredCapital`, `supplierAddress`, `supplierCity`, `supplierCountry`, `postalCode`, `membership`, `memberSince`, `yearsOnPlatform`, `supplierRating`, `reviewCount`, `mainMarkets`, `paymentTerms`, `incoterms`, `leadTime`, `nearestPort`, `exportYear`, `productionLines`, `productionMachines`, `qaQcInspectors`, `rdEngineers`, `oemService`, `odmService`, `ownBrand`, `customizationOptions`, `repeatBuyersRate`, `supplyChainPartners`, `productGroups`, `productCount`, `sampleProducts`.

Audit, contact, and provenance:

`auditAgency`, `auditReportId`, `leadingFactory`, `securedTrading`, `contactPerson`, `contactPosition`, `contactPageUrl`, `aboutUrl`, `showroomUrl`, `logo`, `coverImage`, `companyVideoUrl`, `verifiedFields`, `leadScore`, `searchKeyword`, `page`, `position`, `isSponsored`, `recordType: "supplier"`, `scrapedAt`.

The `leadScore` is computed by this actor from public signals: audit status, membership tier, years on platform, buyer rating, certifications, main markets, export/trade terms, named contact, and facility data. It is not a Made-in-China score.

## Reliability and limits

- **Cookieless and keyless:** public pages only; no login, cookies, or API key.
- **Datacenter fallback:** if the desktop site challenges a datacenter IP, listing/search/category pages switch to the mobile site (`m.made-in-china.com`), while product and supplier pages stay on their supplier subdomains. Mobile detail templates publish fewer fields, so key attributes are also read from visible text and Open Graph metadata.
- **Filters under fallback:** province, membership/audit badges, price, and MOQ filters are applied to records client-side. Site-facet-only filters (business type, R&D capacity, certification, sample availability) may not narrow results when the desktop host is challenged and the mobile fallback is in use.
- **Conditional fields:** Made-in-China suppliers publish different amounts of data. A field is omitted when the source does not publish it, so two suppliers can have different key sets.
- **Contact page:** the public contact page may show a contact person, position, postal code, and showroom. Phone, email, and fax are login-gated and are intentionally not scraped.
- **Price details:** the site's JSON-LD `offers.price` is usually the lowest tier, so this actor derives prices from the visible tier ladder. A product with no published numeric price has no `priceMin`/`priceMax`.
- **Pagination:** product search is deep; `maxItems` and `maxPagesPerQuery` are the cost controls. Listings can re-inject sponsored cards, so records are deduped by canonical URL.
- **No wall-clock cutoffs:** a run stops on `maxItems`, the page cap, source exhaustion, or a repeated page with no new URLs.

## FAQ

**Do I need a Made-in-China account, cookie, or proxy?**
No. The actor uses the public site and works without any user credentials or proxy configuration.

**Can I search several keywords in one run?**
Yes. Put them in `searchQueries`; `maxItems` applies across the whole run. The same multi-input pattern works for categories, product URLs, and supplier URLs.

**How do I scrape a product if I only have its ID?**
Use `productDetails` and put the bare ID in `productUrls`. The actor tries to resolve it through Made-in-China's own search; because product pages live on supplier subdomains, a full product URL is the reliable form and is recommended for scheduled runs.

**Why are some fields missing on a record?**
Made-in-China only publishes certain fields for certain suppliers and templates. Empty fields are omitted rather than filled with `N/A`, `null`, or guessed values.

**What does `priceMin` mean if a product has volume tiers?**
`priceMin` is the lowest price in the visible tier ladder (usually the largest quantity), and `priceMax` is the highest. Use `priceTiers` for the full quantity/price ladder.

**Why is `supplierRating` present on some products but not others?**
The buyer-review widget only appears for suppliers who have received reviews. The underlying source omits it for newer or unreviewed suppliers.

**Can I get supplier phone numbers or email addresses?**
No. Those are login-gated on Made-in-China. The actor returns the public contact page URL, named contact when shown, position, postal code, and showroom instead.

**What is included in `leadScore`?**
It is a sourcing helper computed from audit status, membership, years on platform, rating, certifications, markets, trade terms, contact availability, and facility data. Documented public signals only, never private account data.

**Does browsing a category return the same detail as keyword search?**
The card fields are the same; enable `includeProductDetails` or `includeSupplierProfile` to enrich category rows with detail-page and profile data.

**Is this affiliated with Made-in-China.com?**
No. It is an independent third-party actor that reads publicly available Made-in-China.com pages.
