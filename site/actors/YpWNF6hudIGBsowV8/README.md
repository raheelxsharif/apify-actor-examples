# Google Maps Top List & Ranked Places Extractor

Google Maps ranks the places in a "Top hotels in Paris" search and in every shared list you save. This actor reads that ranked order — directly from Google's public Maps endpoints, without a browser, cookies, an API key, or proxy setup — and writes one row per place.

It handles two kinds of inputs:

- **Top-list queries** such as *Top hotels in Paris*, *Best coffee in Brooklyn*, or *Things to do in Tokyo*.
- **Shared list links** such as `https://www.google.com/maps/placelists/list/...` or a short `maps.app.goo.gl` link. A Maps search URL is accepted too.

## What you get

- The **rank** Google gives each place, plus the query or list it came from.
- Shared-list context: list title and description, list ID, owner, the saver's note, and the saved date.
- Business facts: category, rating, review count, address, phone, website, hours, open/closed status, coordinates, price, and Place IDs.
- Optional detail enrichment: Plus Code, rating histogram, photos, amenities, service options, booking links, hotel stars and amenities, fuel prices, and closure flags.

No record contains `null`, empty strings, or empty arrays. A field Google does not publish for a place is omitted.

## Example row

```json
{
  "recordType": "business",
  "rank": 1,
  "listSource": "Top hotels in Paris",
  "name": "Four Seasons Hotel George V, Paris",
  "categoryName": "Hotel",
  "categories": ["Hotel", "Spa", "Wedding venue"],
  "totalScore": 4.8,
  "reviewsCount": 7414,
  "address": "31 Av. George V, 75008 Paris, France",
  "phone": "+33 1 49 52 70 00",
  "website": "https://www.fourseasons.com/paris/",
  "price": "PKR 663K",
  "priceLevel": "€€€€",
  "highlights": ["Pool", "Spa", "Hot tub", "Free parking"],
  "placeId": "ChIJAcNYmcJv5kcR7EQ_IeMZp8c",
  "featureId": "0x47e66fc29958c301:0xc7a719e3213f44ec",
  "location": { "lat": 48.8688005, "lng": 2.3008313 },
  "googleMapsUrl": "https://www.google.com/maps/place/?q=place_id:ChIJAcNYmcJv5kcR7EQ_IeMZp8c",
  "scrapedAt": "2026-11-06T09:00:00+00:00"
}
```

### Field notes

- `rank` is the position Google returned for that query or list, starting at 1. Filtering never renumbers it.
- `placeId` is the canonical `ChIJ...` ID. A legacy `0x...:0x...` token is kept separately in `featureId`.
- `price` is the amount Google prints on a hotel card, kept verbatim (currency and K/M shorthand included). `priceLevel` is Google's standing band such as `$$$` or a dining price range.
- `highlights` are the short feature pills on a hotel list card. `amenities` is the fuller badge set from the place page.
- `note` and `savedAt` come from the shared list and are present only for list rows. `listTitle` and `listDescription` carry the list's own title and description.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | Select | `topList` | `topList` for ranked queries, `listUrl` for shared-list or Maps URLs. |
| `queries` | String list | – (prefill: `["Top hotels in Paris"]`) | One or more ranked queries. |
| `location` | String | – | Appended to every query when the query does not already mention it. |
| `listUrls` | String list | – | Public shared-list links or Maps search URLs. |
| `maxResults` | Integer | `30` | Maximum places per query or list. |
| `enrichDetails` | Boolean | `true` | Visit each place record for the detail-only fields. |
| `language` | Select | `en` | 70+ Google Maps interface languages. |
| `minRating` / `maxRating` | Number | – | Rating bounds (0–5). |
| `minReviews` / `maxReviews` | Integer | – | Review-count bounds. |
| `openNow` | Boolean | `false` | Keep places Google marks open now. |
| `hasWebsite` / `hasPhone` | Boolean | `false` | Require a published website or phone. |
| `includeClosed` | Boolean | `true` | When off, permanently closed places are dropped. |
| `nameFilter` | String | – | Keep names containing this text. |
| `excludeCategories` | String list | – | Drop categories containing any listed word. |
| `includeCategories` | String list | – | Keep categories containing any listed word. |
| `countryCodes` | String list | – | Keep places in these two-letter countries (FR, US, JP…). |

Saved tasks that still send the old single-value `query` or `listUrl` keep working: the actor maps those keys in code.

## Examples

### Ranked hotels

```json
{
  "mode": "topList",
  "queries": ["Top hotels in Paris"],
  "maxResults": 10,
  "enrichDetails": true,
  "language": "en"
}
```

### Fast card-only run

```json
{
  "mode": "topList",
  "queries": ["Best sushi restaurants in Tokyo"],
  "maxResults": 25,
  "enrichDetails": false,
  "language": "ja"
}
```

### A shared list

```json
{
  "mode": "listUrl",
  "listUrls": ["https://www.google.com/maps/placelists/list/oMnWVsXSvFX9Bh2O5-71TApDgffWvg"],
  "maxResults": 5000,
  "enrichDetails": false
}
```

## Use cases

- Travel research: compile ranked hotels, restaurants and attractions per city.
- Local SEO: compare how a category ranks across markets.
- Lead generation: export name, rating, phone, website and address from a shortlist.
- Content: turn shared "best of" lists into structured tables.
- Site selection: measure the density and quality of a category in an area.

## FAQ

**Does it need cookies, a login or an API key?**
No. Every mode reads public Google Maps data.

**Do I need to configure a proxy?**
No. The actor chooses its connection automatically. There is no proxy field to fill in.

**How many places can a query return?**
It varies by query and city. Broad categories commonly expose 150–450 ranked places; narrow queries expose fewer. The actor follows Google's pages until `maxResults`, the end of the result window, or the source's own stop signal.

**How many places can a shared list have?**
The public list endpoint returns the whole list in one response. This actor requests up to 5,000 places per list; that is the practical source window for very large lists.

**Why did I get fewer rows than `maxResults`?**
Google's result window ended, the list was shorter, or filters removed rows. The run status says which case applies and how many rows the filters skipped.

**What does `enrichDetails` add?**
Plus Code, rating histogram, photos, amenities, service options, booking links, hotel details, fuel prices and closure flags. It costs one extra request per place; turn it off when the list-card fields are enough.

**Are sponsored places included?**
No. Google can print paid hotel cards at the top of a feed. They are excluded, so `rank` follows the organic list, as in the previous version.

**Are `popularTimesSummary` and `menuLink` filled?**
`popularTimesSummary` is not published by the public Maps feed used here, so it is not emitted. `menuLink` appears only when a place's detail data exposes a menu URL, which is rare. Use the dedicated popular-times or menu actor when you need those fields.

**Does it support non-English Maps?**
Yes. Pick any language in the dropdown; names, categories, addresses and hours are localized by Google.

**What happens when Google blocks a request?**
The actor retries and escalates its connection automatically. If no result view ever arrives, the run status says it was an upstream block or degraded response — never "no places found".

**Is this affiliated with Google?**
No. It is an independent tool that reads publicly available Google Maps pages. You are responsible for using the data lawfully and consistently with Google's terms.
