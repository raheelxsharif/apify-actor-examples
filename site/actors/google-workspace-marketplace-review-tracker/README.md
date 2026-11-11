# Google Workspace Marketplace Review Tracker

Follow the public review stream for Google Workspace Marketplace add-ons. Point the actor at one
or more app IDs or app URLs, choose the review language and filters, and get a clean dataset of
reviewer names, stars, review text, dates, helpful votes and developer replies. Schedule it with a
state key to receive only reviews that appeared or changed since the previous run.

No Google account, API key, cookie or proxy is needed.

## What you can do

| Task | How |
|---|---|
| Watch your app and competitors | Add each numeric App ID or full Marketplace app URL to `appIds` |
| Discover popular apps by keyword | Set `mode` to search and add keywords to `searchQueries` |
| Read reviews in another language | Pick a language, or keep all supported languages |
| Find complaints | Set a maximum star rating and a review-text keyword |
| Collect recent feedback only | Set a relative `sinceDate` such as `30 days` |
| Alert on new or changed reviews | Use `outputMode` `new` or `changed` with a `stateKey` |
| Re-baseline a monitor | Turn on `resetState` for one run |

## Inputs

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | select | `appReviews` | `appReviews` watches the app list; `search` ranks the public Marketplace index by keyword. |
| `appIds` | stringList | `["1008170693301"]` | Numeric IDs or full app URLs. One app per item. |
| `searchQueries` | stringList | `[]` | Keywords for search mode, e.g. `mail merge`. |
| `maxApps` | integer | `10` | Top matches to track per search keyword. |
| `minAppRating` | number | `0` | Skip apps below this average rating. |
| `minAppReviews` | integer | `0` | Skip apps with fewer total reviews. |
| `sortBy` | select | `Newest` | `Newest` or `MostHelpful` from the server; legacy `MostRelevant`, `HighestRated` and `LowestRated` are also accepted (rating sorts are computed over the collected reviews). |
| `reviewLanguage` | select | `all` | `all` or one of 28 supported language codes. |
| `minRating` | integer | – | Lowest star rating to include (1–5). |
| `maxRating` | integer | – | Highest star rating to include (1–5). |
| `sinceDate` | string | – | ISO date or relative period (`30 days`, `2 weeks`, `6 months`). |
| `untilDate` | string | – | Latest review date, inclusive (YYYY-MM-DD). |
| `reviewKeyword` | string | – | Keep reviews containing this phrase, case-insensitive. |
| `maxReviewsPerApp` | integer | `50` | Per-app cap after filters (1–2000). |
| `maxItems` | integer | `50` | Total rows for the run (1–5000). |
| `outputMode` | select | `all` | `all`, `new` (only unseen reviews) or `changed` (new plus helpful/rating/reply changes). |
| `trackChanges` | boolean | `false` | Add `isNew`/`changeTypes` and save a snapshot while returning all rows. |
| `stateKey` | string | – | Snapshot name; use one per schedule or watchlist. Derived when blank. |
| `resetState` | boolean | `false` | Clear the snapshot before the run. |

### How to find an App ID

Open the app in the Marketplace. The numeric part after the final slash is the App ID:

```
https://workspace.google.com/marketplace/app/mailmeteor_mail_merge_for_google_sheets/1008170693301
                                                                                       ^^^^^^^^^^^^^
```

Both that full URL and the bare `1008170693301` are accepted. If you only have the ID, the actor
resolves the listing automatically.

## Output

Every row has `recordType` `workspace_review` and these fields when the source provides them:

| Field | Meaning |
|---|---|
| `reviewId` | Marketplace review identifier. |
| `appId`, `appName`, `appUrl` | The reviewed app and its listing URL. |
| `reviewerName`, `reviewerAvatarUrl` | Public reviewer name and profile image. |
| `rating` | Star rating, 1–5. |
| `reviewText` | Written review text; omitted for rating-only reviews. |
| `reviewDate`, `reviewTimestamp` | UTC date and full ISO timestamp. |
| `helpfulCount` | Helpful votes; omitted when zero, matching earlier versions of this actor. |
| `developerReply`, `developerReplyDate`, `developerReplyAuthor`, `developerReplies` | Latest developer reply plus every reply thread entry (`replyId`, `author`, `text`, `date`, `timestamp`). |
| `reviewLanguage` | Language slice the review came from (for example `de`). |
| `appDeveloper`, `appRating`, `appReviewCount`, `appInstallCount`, `appInstallsLabel` | App context from the listing. |
| `appPricingModel`, `appIconUrl`, `appWorksWith`, `appLastUpdated` | Pricing, icon, Google products and listing-update date. |
| `searchQuery`, `appRankInSearch` | Present in search mode. |
| `isNew`, `changeTypes` | Present when monitor tracking is on. |
| `scrapedAt` | UTC timestamp of this run. |

Empty values are omitted at every level; a row never contains `null`, `""`, `[]` or `{}`.

## Monitoring new reviews

1. Set `outputMode` to `new` or `changed`.
2. Give the watchlist a memorable `stateKey`, for example `my-app-reviews`.
3. Schedule the actor daily. The first run is the baseline and returns the matching reviews.
4. Later runs load the saved review fingerprints and emit only unseen reviews (`new`), or unseen
   reviews plus reviews whose helpful count, rating or reply count moved (`changed`).

The snapshot lives in a named key-value store, so separate scheduled runs share it. It holds up to
50,000 reviews; if a very large history is trimmed, the oldest entries are dropped first.
`resetState` clears the snapshot without deleting the dataset.

## Search mode

Search ranks the Marketplace's public top-100 app index, then collects reviews for the best
keyword matches. It is a discovery shortcut, not a full catalogue search: niche add-ons that are
not in that index must be added by ID or URL in appReviews mode. The run status says when no app
matched.

## Languages and pagination

Reviews are served in 10-row pages per language. The actor requests the pages itself, follows the
language totals, deduplicates by review ID, and stops on the per-app cap. Sparse filters scan at
most `max(10, maxReviewsPerApp)` pages per language, so raise `maxReviewsPerApp` to look deeper
into the history. With `reviewLanguage: all`, it visits the supported language slices in turn, so
a single language cannot use up the whole cap. The supported list covers English, German, Spanish,
French, Portuguese (Brazil), Italian, Russian, Japanese, Korean, Chinese (Simplified and
Traditional), Indonesian, Vietnamese, Dutch, Polish, Turkish, Arabic, Hindi, Thai, Hebrew,
Ukrainian, Swedish, Danish, Finnish, Norwegian, Czech, Hungarian and Filipino.

## FAQ

**Does this need a Google login or API key?**
No. It reads the same public review data the Marketplace shows to signed-out visitors.

**Why does the app page show more ratings than I receive?**
The rating count includes every star rating, including rating-only submissions and every language.
Written reviews are also split by language. Raise `maxReviewsPerApp` and keep `reviewLanguage` on
`all` for the widest coverage.

**Can I sort by highest or lowest rating?**
The Marketplace itself only offers Newest and Most Helpful. Old saved tasks that still send
`HighestRated` or `LowestRated` keep working: the actor fetches recent reviews and sorts the
collected set by stars. The status message explains the fallback.

**Why are some reviews missing text?**
Some visitors leave only a star rating. Those rows keep the rating and omit `reviewText`.

**Are developer replies included?**
Yes. Replies appear in the public review list and are attached to their review, including the
reply author and date. A reply to a review outside the fetched window is not attached; raise
`maxReviewsPerApp` or use a recent date filter to widen the window.

**What if an app has no reviews in the chosen language?**
The run moves to the next language in `all` mode and finishes cleanly with zero rows for that
language slice. A completely empty result is reported in the status message, not as a failure.
