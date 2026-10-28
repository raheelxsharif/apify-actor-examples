# Goodreads Book, Author & Listopia List Scraper

Pull **Goodreads** book data, reader reviews, editions, quotes, author profiles, series, Listopia rankings, shelf and genre charts, similar books, and new-release charts into one dataset. Look up books by URL, ID, ISBN, or keyword, or walk an entire author, series, or Listopia list. Every run is cookieless: no Goodreads login, account, or API key is needed.

## What this actor does

- **Fifteen modes:** `books`, `search`, `isbns`, `authors`, `series`, `lists`, `shelves`, `genres`, `reviews`, `editions`, `quotes`, `similar`, `bookLists`, `newReleases`, and `auto`.
- **Books by URL, ID, or ISBN** with ratings, reviews, editions, awards, characters, places, series, social counters, and buy links.
- **Reader reviews** with full text, rating, date, reviewer profile, likes, comments, shelves, language, and sort filters.
- **Discovery rows** for search results, author bibliographies, series reading order, Listopia ranks, shelves, genres, similar books, and monthly or yearly release charts.
- **Filters** for minimum rating, ratings count, first-published year range, language, genre, and author, plus per-book review, edition, quote, similar-book, and list caps.
- **Clean records:** empty fields are omitted at every depth, never returned as `null`, `""`, `[]`, or `{}`.

## Output

Every record has a `recordType` and a `scrapedAt` ISO 8601 UTC timestamp. Empty fields are omitted.

### Book (`recordType: "book"`)

- Identity: `bookId`, `kcaId`, `title`, `titleComplete`, `originalTitle`, `url`, `workId`, `workKcaId`, `workUrl`, `coverImage`
- Description: `description` (plain text), `descriptionHtml`
- Contributors: `authors[]`, `primaryAuthor`, `authorUrls[]`, `contributors[]` (`name`, `authorId`, `url`, `role`, `isGoodreadsAuthor`)
- Series: `series` (`name`, `position`, `url`, `id`, `kcaId`), `allSeries[]`
- Edition: `isbn`, `isbn10`, `isbn13`, `asin`, `pagesCount`, `format`, `publisher`, `language`, `languageCode`, `publishedYear`, `publishedAt`, `firstPublishedYear`, `firstPublishedAt`
- Ratings: `averageRating`, `ratingsCount`, `reviewsCount`, `ratingsDistribution` (1–5 stars), `reviewsByLanguage`
- Discovery: `genres[]`, `awards[]`, `characters[]`, `places[]`, `editionsCount`, `editionsUrl`, `quotesCount`, `questionsCount`, `topicsCount`, `currentlyReadingCount`, `wantToReadCount`
- Commerce: `kindleUrl`, `buyLinks[]`

### Review (`recordType: "review"`)

`reviewId`, `bookId`, `workId`, `rating`, `preciseRating`, `reviewText`, `reviewTextHtml`, `spoiler`, `likesCount`, `commentsCount`, `createdAt`, `updatedAt`, `reviewer` (`id`, `name`, `url`, `imageUrl`, `isAuthor`, `followersCount`, `textReviewsCount`), `shelves[]`

### Edition (`recordType: "edition"`)

`editionId`, `bookId`, `sourceBookId`, `workId`, `title`, `format`, `publishedYear`, `publishedAt`, `publisher`, `pagesCount`, `authors[]`, `primaryAuthor`, `isbn10`, `isbn13`, `asin`, `language`, `languageCode`, `averageRating`, `ratingsCount`, `coverImage`, `url`. For editions, `bookId` is the edition's own Goodreads id; `sourceBookId` is the book supplied in the input.

### Quote (`recordType: "quote"`)

`quoteId`, `quoteText`, `authorName`, `bookTitle`, `workId`, `workUrl`, `tags[]`, `likesCount`, `quoteUrl`, `imageUrl`, `sourceType`, `sourceValue`

### Author (`recordType: "author"`, with `includeMetadata`)

`authorId`, `name`, `authorUrl`, `photoUrl`, `description`, `born`, `died`, `website`, `twitter`, `genres[]`, `influences[]`, `series[]` (`seriesId`, `name`, `url`, `bookCount`, `authors`, `averageRating`, `ratingsCount`)

### Series (`recordType: "series"`, with `includeMetadata`)

`seriesId`, `name`, `seriesUrl`, `description`, `primaryWorksCount`, `totalWorksCount`

### Listopia list (`recordType: "list"`)

`listId`, `name`, `listUrl`, `description`, `bookCount`, `voterCount`, `createdDate`, `creator` (`name`, `url`), `sourceBookId`. Records emitted by `bookLists` carry the list id, name, URL, book count, voter count, and `sourceBookId`; the full description, creation date, and creator come from the `lists` mode with `includeMetadata`.

### Listing rows

- `searchResult` (`query`) and `authorWork` (`authorId`): `bookId`, `title`, `url`, `coverImage`, `authors[]`, `primaryAuthor`, `series`, `averageRating`, `ratingsCount`, `reviewsCount`, `publishedYear`, `editionsCount`
- `seriesBook` (`seriesId`, `position`) and `listEntry` (`listId`, `rank`, `score`, `voterCount`)
- `shelfBook` (`shelfName`, `shelfCount`), `genreBook` (`genre`, `group`)
- `similarBook` (`sourceBookId`)
- `newRelease` (`year`, `month`, `rank`, `count`)

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `auto` | `auto`, `books`, `search`, `isbns`, `authors`, `series`, `lists`, `shelves`, `genres`, `reviews`, `editions`, `quotes`, `similar`, `bookLists`, `newReleases` |
| `bookUrls` | array | – | Book URLs, IDs, or ISBNs (books, reviews, editions, similar, bookLists) |
| `searchQueries` | array | – | Titles, authors, or keywords (search) |
| `isbns` | array | – | ISBN-10 or ISBN-13 values (isbns) |
| `authorUrls` | array | – | Author URLs or IDs (authors) |
| `seriesUrls` | array | – | Series URLs or IDs (series) |
| `listUrls` | array | – | Listopia URLs or IDs (lists) |
| `shelfNames` | array | – | Shelf names such as `mystery` (shelves) |
| `genreNames` | array | – | Genre slugs such as `fantasy` (genres) |
| `maxItems` | integer | `50` | Hard cap on every emitted record |
| `detailLevel` | string | `full` | `full` book records or `listing` rows |
| `includeMetadata` | boolean | `false` | Add author, series, or Listopia summary records |
| `includeRelated` | boolean | `false` | In `auto` mode, also collect reviews, editions, similar books, and lists for each book URL |
| `minRating` | number | – | Drop books rated below this value (decimals allowed) |
| `minRatingsCount` | integer | – | Drop books with fewer ratings |
| `publishYearMin` | integer | – | Drop books first published before this year |
| `publishYearMax` | integer | – | Drop books first published after this year |
| `language` | string | – | Keep books whose language starts with this text |
| `containsGenre` | string | – | Keep books with a genre containing this text |
| `containsAuthor` | string | – | Keep books with an author containing this text |
| `maxReviewsPerBook` | integer | `30` | Reviews per book (reviews) |
| `reviewSort` | string | `DEFAULT` | `DEFAULT`, `NEWEST`, `OLDEST` (reviews) |
| `reviewLanguage` | string | `all` | Review language code (reviews) |
| `reviewRatingMin` | integer | – | Minimum review star rating (reviews) |
| `reviewRatingMax` | integer | – | Maximum review star rating (reviews) |
| `excludeSpoilers` | boolean | `false` | Skip reviews marked as spoilers (reviews) |
| `maxEditionsPerBook` | integer | `20` | Editions per book (editions) |
| `quoteTags` | array | – | Quote tag pages such as `love` (quotes) |
| `quoteAuthorUrls` | array | – | Author quote pages (quotes) |
| `quoteWorkUrls` | array | – | Work quote pages (quotes) |
| `maxQuotesPerSource` | integer | `30` | Quotes per tag, author, or work (quotes) |
| `maxSimilarPerBook` | integer | `20` | Similar books per input book (similar) |
| `maxListsPerBook` | integer | `20` | Lists per input book (bookLists) |
| `releaseYear` | integer | `2026` | Release chart year (newReleases) |
| `releaseMonth` | string | `all` | `all` or `1`–`12` (newReleases) |

### Example: book detail by URL

```json
{
  "mode": "books",
  "bookUrls": ["https://www.goodreads.com/book/show/4671.The_Great_Gatsby"],
  "maxItems": 5
}
```

### Example: newest English reviews for two books

```json
{
  "mode": "reviews",
  "bookUrls": ["4671", "9780743273565"],
  "maxReviewsPerBook": 100,
  "reviewSort": "NEWEST",
  "reviewLanguage": "en",
  "reviewRatingMin": 4,
  "excludeSpoilers": true
}
```

### Example: Listopia rows plus a list summary

```json
{
  "mode": "lists",
  "listUrls": ["https://www.goodreads.com/list/show/1.Best_Books_Ever"],
  "detailLevel": "listing",
  "includeMetadata": true,
  "maxItems": 500
}
```

### Example: monthly release chart

```json
{
  "mode": "newReleases",
  "releaseYear": 2026,
  "releaseMonth": "10",
  "maxItems": 30
}
```

### Example: full author bibliography

```json
{
  "mode": "authors",
  "authorUrls": ["https://www.goodreads.com/author/show/1077326.J_K_Rowling"],
  "minRating": 4,
  "maxItems": 60
}
```

## How it works

Fill in one or more input arrays and pick a mode, or leave `mode` on `auto` to run every non-empty input in one pass. The actor reads Goodreads' public book pages, listing pages, and the same public data service the signed-out website uses, then streams records to the dataset as they are ready. Use `detailLevel: "listing"` when you only need ranks and row metadata; use `full` when you need complete book records. `maxItems` is a hard cap across the whole run, so a large request stops exactly at your limit rather than over-delivering. Set `includeRelated` when an auto run should also collect reviews, editions, similar books, and lists for its book URLs.

## Usage cost

You pay only for Apify compute time on the selected memory setting; there is no per-record charge from this actor. Listing modes are the cheapest because they skip per-book detail calls. Review and full-detail runs cost more because each book requires additional requests.

## FAQ

**Do I need a Goodreads account, cookies, or an API key?**
No. The actor reads public data only and works without any credentials.

**Do I need to configure a proxy?**
No. Requests go direct first. If Goodreads challenges an HTML request, the actor retries and then tries a fresh Apify AUTO datacenter session when your account has that proxy access; book pages also have a `.xml` fallback. GraphQL and autocomplete run without a user proxy.

**What is the difference between `full` and `listing`?**
`full` returns complete book records with description, ratings, awards, and edition data. `listing` returns one lightweight row per result with the rank, title, author, rating, and URL, which is faster for large lists.

**Why are reviews missing some likes, comments, or shelves?**
Goodreads itself omits those counters on some deleted or restricted reviews. The actor keeps the review text and rating and leaves the missing fields out.

**Why is a book missing its ISBN or publisher?**
Not every Goodreads edition records those fields. The actor never invents values; it omits the field when the source does not publish it.

**How far can search, Listopia, quotes, and shelves go?**
Goodreads caps search at about 2,000 results, Listopia at 10,000 entries per list, quote pages at 3,000 quotes per source, and popular shelf pages at 50 books. Genre pages expose the three curated groups shown on the page. The actor reports these limits in the run status instead of silently under-delivering.

**Does `maxItems` include summary records?**
Yes. Every record counts, including review, edition, quote, author, series, and list records.

**Which memory setting does the actor use?**
The actor requests 1,024 MB and streams records instead of buffering a run. Account-level defaults can raise the effective run memory (often 4,096 MB on Apify accounts), so compute cost follows the memory Apify actually assigns.

**Are the links and covers usable elsewhere?**
Book, cover, author, and buy links are the public URLs Goodreads publishes. A very old or rare cover may no longer be served by Goodreads itself.

**Is this affiliated with Goodreads or Amazon?**
No. This is an independent third-party actor that reads Goodreads' public data.

## Notes and limitations

- Search is capped by Goodreads at roughly 2,000 results per query. Split broad topics into narrower queries for deeper coverage.
- Listopia lists stop at 10,000 entries and quote pages at 3,000 quotes per source; both are site limits.
- Popular shelves return the 50 books Goodreads shows; there is no deeper shelf pagination. Use Listopia or genre pages for larger discovery sets.
- Genre pages expose the new-release, most-read, and popular groups shown on the page, not an unbounded catalog.
- Deleted reviews and incomplete editions can lack optional counters or identifiers; those fields are omitted.
- If Goodreads challenges HTML with its AWS WAF page, the actor retries, then attempts an Apify AUTO proxy session; if the challenge persists, it uses book `.xml` and search autocomplete fallbacks where available and reports the challenge in the run status. GraphQL book, review, edition, similar, and list data is not WAF-gated.
- The `.xml` book fallback can resolve to the work's primary edition, so its `bookId` may differ from the edition you requested and it carries fewer social/related fields. The actor uses GraphQL whenever that endpoint answers.
- Data is live at run time; ratings, review counts, and chart positions change continuously.
