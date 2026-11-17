# RateMDs Doctor, Practice & Patient Review Finder

RateMDs publishes patient ratings, reviews and practice details for doctors and clinics across the US and Canada. This actor searches that directory by name, keyword, specialty, city or state, browses practices, or fetches profiles you already know — without cookies, logins or API keys.

## Modes

| Mode | What it returns |
| --- | --- |
| `search` | Doctors matching a name/keyword, optionally narrowed by specialty, city, state, country and profile filters. |
| `bySpecialty` | Every doctor listed under a medical specialty, optionally narrowed by location. |
| `doctorProfile` | One full doctor profile from a slug or URL, including reviews. |
| `byUrl` | A batch of doctor profile URLs in one run. |
| `practiceSearch` | Practices (clinics) in a city/state, with optional per-practice details. |
| `practiceProfile` | One or more practice profiles from clinic URLs, including doctors and reviews. |

## Output per doctor

| Field | Description |
| --- | --- |
| `recordType` | `doctor` or `practice`. |
| `doctorId` | RateMDs internal doctor ID. |
| `name` / `slug` / `url` | Doctor name, URL slug and absolute profile URL. |
| `specialty` / `specialtySlug` | Human-readable and slug form of the specialty. |
| `gender` | `M` or `F` when RateMDs publishes it. |
| `verified` | Whether RateMDs marks the profile verified. |
| `rating` / `reviewCount` | Average star rating (1.0–5.0) and total review count. |
| `ratingBreakdown` | `staff`, `punctuality`, `helpfulness`, `knowledge` sub-ratings when available. |
| `photoUrl` | Absolute profile photo URL. |
| `address` / `suite` / `city` / `state` / `country` / `postalCode` | Primary practice location. |
| `latitude` / `longitude` | Practice coordinates when published. |
| `phone` / `website` | Practice contact details. |
| `clinicName` / `clinicId` / `clinicUrl` | Primary practice name, ID and RateMDs page. |
| `acceptsNewPatients` / `acceptsVirtualVisits` / `onlineScheduling` | Availability flags. |
| `sampleReview` | A sample review shown on the listing card. |
| `bio` / `languages` / `expertises` / `education` | Profile-only details when published. |
| `recentReviews` | Recent RateMDs patient reviews (`id`, `date`, `rating`, `text`, `authorLabel`, `appointmentType`, sub-ratings, `helpfulVotes`). |
| `partnerReviews` | Recent "Our Partners" reviews (same shape, `source: "partner"`). |
| `ratemdsReviewCount` / `partnerReviewCount` | Per-source review totals. |
| `scrapedAt` | ISO timestamp of capture. |

## Output per practice

`recordType`, `practiceId`, `practiceName`, `slug`, `practiceUrl`, `rank`, `locationLabel`, `address`, `suite`, `city`, `state`, `country`, `postalCode`, `phone`, `website`, `latitude`, `longitude`, `rating`, `reviewCount`, `ratingBreakdown` (`cleanliness`, `facilities`, `services`, `value`), `doctors` (id, name, URL, specialty, rating), `doctorCount`, `recentReviews`, `scrapedAt`.

Fields a profile does not publish are omitted — never filled with `null`, `0` or a placeholder.

## Input

| Field | Type | Default | Description |
| --- | --- | --- | --- |
| `mode` | Select | `search` | See the mode table. |
| `searchQuery` | Text | `cardiologist` | Doctor name, keyword or condition. |
| `specialty` | Select (66 options) | Any | Specialty browse/filter. |
| `location` | Text | – | Legacy free-text location (`New York`, `Toronto, ON`). |
| `state` / `city` | Text | – | Structured location (e.g. `ny`, `new-york`). |
| `country` | Select | Any | `us`, `ca`, `au`, `in`, `za`, `uk`. |
| `gender` | Select | `any` | `F`, `M`. |
| `verified` | Boolean | `false` | Verified profiles only. |
| `acceptingNewPatients` | Boolean | `false` | Accepting new patients only. |
| `acceptingVirtualVisits` | Boolean | `false` | Virtual visits only. |
| `onlineScheduling` | Boolean | `false` | Online booking only. |
| `openNow` | Boolean | `false` | Practices open now. |
| `doctorSlug` | Text | – | Doctor slug or URL (`doctorProfile`). |
| `profileUrls` | String list | `[]` | Doctor URLs (`byUrl`). |
| `practiceUrls` | String list | `[]` | Clinic URLs (`practiceProfile`). |
| `includeReviews` | Boolean | `false` | Fetch reviews (adds requests). |
| `includePartnerReviews` | Boolean | `false` | Fetch "Our Partners" review pages. |
| `includePracticeDetails` | Boolean | `false` | Fetch each practice page in `practiceSearch`. |
| `maxReviewPages` | Integer | `1` | Review pages per profile (10 reviews/page). |
| `maxPages` | Integer | `1000` | Listing pages per run. |
| `maxItems` | Integer | `20` | Records to emit. |

### Examples

```json
{"mode": "search", "searchQuery": "Dr. Jane Doe", "state": "ny", "includeReviews": true, "maxItems": 5}
```

```json
{"mode": "bySpecialty", "specialty": "dermatologist", "state": "ca", "city": "los-angeles", "maxItems": 50}
```

```json
{"mode": "practiceSearch", "state": "sc", "city": "north-charleston", "includePracticeDetails": true, "maxItems": 10}
```

```json
{"mode": "doctorProfile", "doctorSlug": "45808/Dr-Paul-Natterson-Santa+Monica-CA.html", "includeReviews": true, "maxReviewPages": 3}
```

## Pagination and limits

Doctor listings normally hold 10 results per page and RateMDs caps a single query at 1,000 pages (about 10,000 doctors). When a query is larger than that, split it by state, city, specialty or one of the profile filters. Practice listings hold 10 practices per page; `locations_count` on the page gives the true total. The actor stops at `maxItems`, when the source runs out, or when `maxPages` is reached — it never stops on a wall-clock timer.

## How it works

RateMDs serves server-rendered pages with structured `window.DATA` payloads. The actor reads those payloads directly instead of brittle CSS selectors. When RateMDs returns its JavaScript proof-of-work challenge (HTTP 202), the actor solves it locally, stores the bypass cookie and retries. Requests are paced and use Apify Proxy sessions when the site rate-limits or blocks the direct connection; no proxy or cookie setup is required from you.

## FAQ

**Do I need a RateMDs account?** No. The actor reads the public pages shown to any visitor.

**Why are some fields missing?** RateMDs does not publish a website, phone, photo or sub-rating for every profile; those fields are omitted for that record.

**How many reviews can I get?** The profile page embeds the most recent page; `includeReviews` plus `maxReviewPages` walks the public review endpoint for more. `includePartnerReviews` adds partner reviews where present.

**Can I paste a full profile URL?** Yes. Both doctor and clinic URLs are accepted in the profile modes.

**Why do I see fewer rows than the site count?** A single query is capped by RateMDs at 1,000 pages. Partition by location/specialty/filter to reach the rest.

**Is this affiliated with RateMDs?** No. This is an independent tool and is not affiliated with, endorsed by, or sponsored by RateMDs.
