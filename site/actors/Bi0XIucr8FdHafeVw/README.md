# TrustRadius Products, Reviews & Pricing Scraper

Collect the evidence B2B buyers use on **TrustRadius**: product profiles, thousands of verified reviews, feature-by-feature scores, pricing plans, FAQs, alternatives and side-by-side comparisons. Choose a mode, point it at a product, keyword, category or comparison, and get clean structured records instead of opening dozens of browser tabs.

## What this actor does

- **Four modes:** `product` for profile pages, `reviews` for the review archive, `discover` for category and keyword listings, and `compare` for two-product comparisons
- **Complete review filtering:** star rating, company size, reviewer country, buying role, job type, department, minimum experience, recency and four sort orders
- **Commercial context:** vendor, starting price, plan names and billing terms, free version/trial flags, feature scorecards, awards and competitor cards
- **Review depth:** reviewer name, job title, department, company, company size, experience, LinkedIn profile when public, pros, cons, use cases, likelihood to recommend, plus optional ROI and full Q&A
- **Reliable pagination:** reviews arrive 10 per page, category browsing follows TrustRadius's 1,000-product ceiling, and keyword discovery can walk the full directory result set
- **Empty fields are omitted** at every level, so records never contain null, blank strings or empty lists

## Output per record

The dataset contains one record per product, review, discovery card or comparison.

### Product records (`recordType: "product"`)

- `sourceUrl`, `canonicalUrl`, `slug`, `title`, `description`, `about`
- `applicationCategory`, `category`, `vendorName`, `vendorUrl`, `logoUrl`
- `ratingValue`, `ratingNormalized`, `ratingCount`, `reviewCount`, `reviewPageCount`, `ratingBreakdown[]`
- `price`, `priceCurrency`, `pricingPlans[]`, `hasFreeVersion`, `hasFreeTrial`
- `awards[]`, `featureList[]`, `topReviews[]`, `positiveNotes[]`, `negativeNotes[]`
- `topPerformingFeatures[]`, `areasForImprovement[]`, `featureGroups[]`
- `technicalDetails{}`, `faqs[]`, `screenshots[]`, `videos[]`, `mediaUrls[]`
- `alternatives[]`, `relatedProducts[]`
- `recordType`, `scrapedAt`

### Review records (`recordType: "review"`)

- `reviewUrl`, `productName`, `productSlug`, `title`, `rating`, `ratingNormalized`, `publishedAt`
- `reviewSource`, `incentivized`, `reviewer{}` with name, job title, department, company, company size, experience, profile URL, LinkedIn URL and verified flag
- `useCase`, `pros[]`, `cons[]`, `likelihoodToRecommend{}`, `sections[]`
- `roi[]`, `alternativesConsidered`, `otherSoftwareUsed`
- `productAggregate{}`, `recordType`, `scrapedAt`

### Discovery records (`recordType: "productCard"`)

- `source` (`search` or `category`), `rank`, `productName`, `slug`, `productUrl`, `canonicalUrl`
- `category`, `rating`, `ratingNormalized`, `ratingCount`, `reviewCount`, `description`, `tags[]`, `logoUrl`
- `recordType`, `scrapedAt`

### Comparison records (`recordType: "compare"`)

- `compareUrl`, `title`, `products[]` with name, slug, URLs, rating, review count, starting price, summary, most-used-by segment and logo
- `pricing` plan names, prices and billing terms per product, plus free-version/free-trial flags
- `featureGroups[]` with group scores and per-feature scores for both products
- `alternatives[]`, `userRatings[]`, `communityPulse[]`
- `recordType`, `scrapedAt`

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `product` | `product`, `reviews`, `discover` or `compare` |
| `productUrls` | array | `["https://www.trustradius.com/products/asana"]` | Product URLs, review URLs or bare slugs; compare mode also accepts a comparison URL or two products |
| `searchQueries` | array | `[]` | Keywords for the directory search (mode=discover) |
| `categoryUrls` | array | `[]` | Category URLs or slugs (mode=discover) |
| `maxItems` | integer | `25` | Cap on emitted rows (1–5,000) |
| `maxReviewsPerProduct` | integer | `100` | Reviews per product; `0` means the whole archive (mode=reviews) |
| `reviewSort` | string | `newest` | `newest`, `oldest`, `highestRated` or `lowestRated` |
| `starRatings` | array | `[]` | Keep ratings 1–5 |
| `companySizes` | array | `[]` | `small`, `medium`, `large` |
| `countries` | array | `[]` | ISO3 reviewer countries such as `USA`, `CAN`, `GBR` |
| `reviewerRoles` | array | `[]` | `user`, `manager`, `support`, `decisionMaker`, `consultant`, `implementer` |
| `jobTypes` | array | `[]` | Job types such as `Engineer`, `Manager`, `Director` |
| `departments` | array | `[]` | Departments such as `Marketing`, `Engineering` |
| `minYearsExperience` | integer | `0` | Minimum reviewer experience (0–20) |
| `lastUpdated` | string | `any` | `any`, `month`, `3months`, `6months`, `year` |
| `publishedAfter` | string | `""` | Keep reviews on or after this `YYYY-MM-DD` date |
| `includeReviewDetails` | boolean | `false` | Open each review for ROI, alternatives and full Q&A |
| `includeProductDetails` | boolean | `true` | Collect feature scores, pricing, details, FAQs and alternatives with product records |

### Example: product profile with every detail page

```json
{
  "mode": "product",
  "productUrls": ["https://www.trustradius.com/products/asana"],
  "includeProductDetails": true,
  "maxItems": 1
}
```

### Example: lowest-rated reviews from small companies

```json
{
  "mode": "reviews",
  "productUrls": ["asana"],
  "maxReviewsPerProduct": 50,
  "starRatings": [1, 2],
  "companySizes": ["small"],
  "reviewSort": "oldest"
}
```

### Example: browse a category and a keyword

```json
{
  "mode": "discover",
  "categoryUrls": ["crm"],
  "searchQueries": ["project management"],
  "maxItems": 200
}
```

### Example: compare two products

```json
{
  "mode": "compare",
  "productUrls": ["https://www.trustradius.com/compare-products/asana-vs-monday"]
}
```

## Use cases

- **Software selection** — collect every product in a category with ratings, pricing and feature scores before a shortlist meeting
- **Competitive tracking** — monitor a competitor's rating, review volume, awards and weakest feature areas over time
- **Voice-of-customer research** — export filtered reviews with reviewer firmographics, pros, cons and ROI statements
- **Sales enablement** — build comparison sheets from TrustRadius's own side-by-side data and alternative-product cards
- **Procurement analysis** — compare pricing plans, billing terms, free-trial availability and support details across vendors
- **Market mapping** — discover all software matching a keyword, then enrich the shortlist with full product profiles

## FAQ

**Is this affiliated with TrustRadius?**
No. This is an independent, third-party actor that reads publicly available TrustRadius pages. It is not operated by or affiliated with TrustRadius.

**Which mode should I use?**
Use `product` when you already know the products, `reviews` for the review archive and reviewer detail, `discover` to find products by keyword or category, and `compare` for a two-product page.

**How many reviews can I collect?**
TrustRadius serves 10 reviews per page, and the actor follows pagination until your `maxReviewsPerProduct`, `maxItems` or the end of the matching archive. Some products have thousands of reviews.

**Why does category discovery stop at 1,000 products?**
TrustRadius itself caps a single category listing at 1,000 products. Split large categories with keyword searches or narrower subcategories to reach more products.

**Can I combine several review filters?**
Yes. Star ratings, company sizes, roles, job types, departments, countries, experience and recency can be combined, and the chosen sort order is applied to the result.

**Why are some fields missing from a record?**
TrustRadius does not publish every field for every product or review. Missing values are omitted rather than returned as null, and optional detail pages that do not exist are skipped.

**Does `includeReviewDetails` cost more time?**
Yes. It opens each review to collect ROI, alternatives and the full Q&A. Leave it off when the listing fields are enough.

**Will my saved tasks from the previous version keep working?**
Yes. Existing product URLs, keyword searches, category inputs and page caps are still accepted and mapped to the matching mode. Old proxy settings are ignored because no proxy setup is needed.

**Do I need a proxy or a TrustRadius account?**
No. The actor connects to TrustRadius automatically and only uses publicly visible pages. No cookies, login or proxy configuration are required.
