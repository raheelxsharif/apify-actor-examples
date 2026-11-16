# PitchBook Private Fund Profile Extractor

PitchBook's fund profile pages publish fund size, vintage year, strategy, status and manager details publicly, even though the full platform sits behind a paid subscription. This actor reads those public profile pages directly by URL, by keyword search, or in bulk across PitchBook's fund sitemap, without needing a PitchBook login.

## What this actor does

- Fetches specific fund profiles when you supply their URLs or IDs directly.
- Searches PitchBook by keyword (e.g. "buyout europe") and returns fund profiles up to your item limit; if PitchBook's own search has no fund hits for that keyword, the actor fills the rest of the limit with other fund profiles so the run still returns real data.
- Collects funds in bulk straight from PitchBook's public fund sitemap when no URLs or search query are given.
- Extracts fund size, vintage year, strategy, status, manager, domicile and limited-partner/investment counts from each profile.
- Fields a fund profile doesn't publish (for example no listed social links) are left out of that record.

## Output per fund

| Field | Description |
| --- | --- |
| `name` | Fund name. |
| `profileUrl` | Canonical PitchBook fund profile URL. |
| `description` | Fund description text. |
| `logoUrl` | Fund manager's logo image URL. |
| `socialLinks` | Manager's public social profiles (Twitter/X, LinkedIn, Facebook). |
| `fundStrategy` | Investment strategy (e.g. Buyout, Venture Capital). |
| `fundStatus` | Fund lifecycle status (e.g. Active, Liquidated). |
| `fundSize` | Total committed capital, as published (e.g. "$6.11B"). |
| `vintageYear` | Year the fund was raised. |
| `fundCategory` | Asset class (e.g. Private Equity, Venture Capital). |
| `fundFamily` | Fund family name, when the fund belongs to a numbered series. |
| `fundManager` | Managing firm's name. |
| `fundManagerUrl` | PitchBook profile URL of the managing firm. |
| `fundManagerWebsite` | Managing firm's external website. |
| `fundDomiciles` | Legal domicile jurisdiction(s) of the fund. |
| `nativeCurrency` | Fund's reporting currency. |
| `totalInvestments` | Number of portfolio investments attributed to the fund. |
| `totalLimitedPartners` | Number of limited partners listed for the fund. |
| `streetAddress` / `postalCode` / `city` / `state` / `country` | Manager's address, as published. |
| `scrapedAt` | ISO timestamp of when the record was captured. |

If a profile can't be fetched or parsed, the record instead contains `inputUrl`, `error`, and `scrapedAt`, so failed lookups are still visible in the dataset rather than silently dropped. If an entire run returns zero fund records (for example every attempt was blocked), the dataset instead gets a single diagnostic record describing the run (`mode`, `errorCount`, a status `message` and `scrapedAt`) instead of being left empty.

## Input

| Field | Type | Default | Description |
| --- | --- | --- | --- |
| `profileUrls` | Array of strings | – | Direct PitchBook fund profile URLs or bare IDs (e.g. `11295-73F`). When set, `searchQuery` is ignored and every URL listed is processed regardless of `maxItems`. |
| `searchQuery` | String | `venture capital` | Keyword search used when `profileUrls` is empty. |
| `maxItems` | Integer | 3 | Maximum number of fund records to return. Ignored when `profileUrls` is set. |
| `proxyConfiguration` | Object | Apify Proxy, RESIDENTIAL group | Proxy used to fetch PitchBook pages. A residential proxy is needed because PitchBook blocks repeated requests from the same address. |

### Example: fetch specific fund profiles

```json
{
  "profileUrls": [
    "https://pitchbook.com/profiles/fund/11295-73F",
    "11042-89F"
  ]
}
```

### Example: search by strategy and region

```json
{
  "searchQuery": "growth equity asia",
  "maxItems": 50
}
```

### Example: bulk collection from the fund sitemap

```json
{
  "maxItems": 500
}
```

## Use cases

- **LP due diligence teams** compiling fund size, vintage year and strategy for a shortlist of managers.
- **Deal sourcing teams** building sourcing lists of active funds within a strategy or geography.
- **CRM enrichment** keeping manager contact and website details current without clicking through profiles manually.
- **Market mapping** surveying how many funds a manager has raised across a fund family.
- **Competitive research** comparing fund sizes and vintages across managers in the same strategy.

## FAQ

**Do I need a PitchBook account?**
No. This actor only reads pages PitchBook already publishes to visitors without a login.

**Why does the proxy need to be residential?**
PitchBook limits how many requests it accepts from the same address; a residential proxy is set as the default so profile fetches keep working across a run.

**What data isn't available through this actor?**
Anything PitchBook reserves for paying subscribers — fund IRR/DPI/RVPI/TVPI performance figures, dry powder, individual deal sizes, LP commitment amounts, and full contact phone numbers — isn't published on the public profile page, so it isn't in the output.

**How many funds exist in the bulk sitemap?**
PitchBook's public fund sitemap lists well over 100,000 fund profiles; set `maxItems` to control how many a single run collects.

**Can I search by strategy or geography instead of a fund name?**
`searchQuery` is passed to PitchBook's own fund search, which only indexes a subset of queries; for keywords it doesn't match, the actor fills your remaining `maxItems` with other fund profiles rather than returning nothing. For guaranteed coverage of a specific fund, use `profileUrls` instead.

**How do I tell a fund profile URL apart from an investor profile URL?**
Fund profile IDs end in a capital `F` (e.g. `11295-73F`); investor profile IDs don't have that suffix.

**Is this affiliated with PitchBook?**
No. This is an independent tool and is not affiliated with, endorsed by, or sponsored by PitchBook Data, Inc.
