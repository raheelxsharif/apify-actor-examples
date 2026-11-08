# Google Workspace Marketplace Review Tracker

Monitor user reviews for any Google Workspace Marketplace app. Provide an App ID and get back reviewer names, star ratings, review text, dates, and helpful counts — useful for daily sentiment monitoring, competitor analysis, and reputation tracking.

## What this actor does

- Collects every public review for one Workspace Marketplace app, identified by its numeric App ID
- Filters reviews by minimum star rating and by a "posted on or after" date
- Sorts reviews newest-first, most-relevant, highest-rated-first, or lowest-rated-first
- Returns the "helpful" vote count the Marketplace shows for each review
- Omits `reviewText` or `helpfulCount` when a review has no written text or no helpful votes, rather than returning an empty string or zero

## Output per review

| Field | Type | Description |
|---|---|---|
| `reviewId` | string | Unique review identifier |
| `appId` | string | Numeric App ID |
| `appName` | string | App display name |
| `appUrl` | string | Direct URL to the app listing |
| `reviewerName` | string | Display name of the reviewer |
| `rating` | integer | Star rating (1–5) |
| `reviewText` | string | Full text of the review |
| `reviewDate` | string | ISO-8601 date when the review was posted (YYYY-MM-DD) |
| `helpfulCount` | integer | Number of users who marked the review as helpful |
| `recordType` | string | Always `workspace_review` |
| `scrapedAt` | string | ISO-8601 UTC timestamp of when this record was scraped |

---

## Input options

| Field | Type | Required | Default | Description |
|---|---|---|---|---|
| `appId` | string | Yes | — | Numeric App ID from the Marketplace URL |
| `maxItems` | integer | No | `50` | Max reviews to return (1–500) |
| `sinceDate` | string | No | — | Only return reviews on or after this date (YYYY-MM-DD) |
| `minRating` | integer | No | — | Minimum star rating (1–5). Leave blank for all |
| `sortBy` | select | No | `Newest` | Sort order: Newest, MostRelevant, HighestRated, LowestRated |

### How to find the App ID

The App ID is the numeric part at the end of a Google Workspace Marketplace URL:

```
https://workspace.google.com/marketplace/app/mailmeteor_mail_merge_for_gmail/1008170693301
                                                                              ↑
                                                                       This is the App ID
```

---

## Example output record

```json
{
  "reviewId": "06060438314121439815",
  "appId": "1008170693301",
  "appName": "Mailmeteor - Mail merge for Gmail",
  "appUrl": "https://workspace.google.com/marketplace/app/mailmeteor_mail_merge_for_gmail/1008170693301",
  "reviewerName": "Alice Reviewer",
  "rating": 5,
  "reviewText": "Excellent tool for sending personalized emails at scale. Works perfectly with Gmail.",
  "reviewDate": "2026-04-19",
  "helpfulCount": 12,
  "recordType": "workspace_review",
  "scrapedAt": "2026-05-20T10:00:00+00:00"
}
```

---

## Use cases

- **Reputation monitoring** — run daily on your own app to catch new negative reviews quickly.
- **Competitor analysis** — track what users say about competing apps over time.
- **Sentiment trending** — collect reviews with `sinceDate` set to last month and analyze sentiment shifts.
- **Review alerting** — use `minRating: 1` with `sortBy: Newest` to catch only critical 1-star reviews.
- **Product research** — harvest user pain points from competitor reviews to inform your roadmap.

---

## FAQ

**How do I find the App ID for any Workspace Marketplace app?**
Navigate to the app listing and copy the numeric ID from the URL. Example: for the URL `...app/mailmeteor_mail_merge_for_gmail/1008170693301`, the App ID is `1008170693301`.

**Does it require a Google account or API key?**
No. The scraper uses the public marketplace website and requires no authentication or credentials.

**How many reviews can I collect?**
Up to 500 per run. Most popular apps have thousands of reviews.

**What is `sinceDate` used for?**
Set `sinceDate` to `2025-01-01` to only collect reviews posted on or after January 1, 2025. Combine with `sortBy: Newest` to collect only recent reviews for incremental tracking.

**What does `helpfulCount` represent?**
The number of other users who clicked "Helpful" on that review. Reviews with 0 helpful votes will not have this field.

**Why are some reviews missing `reviewText`?**
A small number of reviews consist only of a star rating with no written text. The `reviewText` field is omitted for those.

**What sort orders are available?**
- `Newest` — most recent reviews first (best for daily monitoring)
- `MostRelevant` — Google's relevance ranking
- `HighestRated` — 5-star reviews first
- `LowestRated` — 1-star reviews first (useful for finding complaints)
