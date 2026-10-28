# Goodreads Book, Author & Listopia List Scraper

Goodreads holds ratings, reviews, and metadata for millions of books, but there's no official way to pull that data in bulk. This actor reads Goodreads' own public pages for you — one book at a time by URL, ISBN, or search term, or an entire author, series, Listopia list, shelf, or genre page in a single run.

## What this actor does

- Looks up books directly by URL, free-text search query, or ISBN-10/ISBN-13 (hyphens and spaces are tolerated).
- Walks an author's full book list, a series' full book list, a Listopia list, a popular shelf, or a genre page, and scrapes every book found there.
- Filters results by minimum rating, minimum ratings count, publish year range, language, genre, or author name.
- Optionally adds one summary record per author, series, or list (bio/description and counts) alongside the book records.
- Omits fields a page doesn't publish rather than sending them as `null`, `""`, `[]`, or `{}`.

## Output per record type

Every record carries a `recordType` and a `scrapedAt` (ISO 8601 UTC) timestamp.

### Book (`recordType: "book"`)

- `title`, `url`, `bookId`
- `authors`, `primaryAuthor`, `authorUrls`
- `description` — plain text, HTML stripped
- `isbn`, `isbn10`, `isbn13` — whichever identifiers Goodreads publishes for that edition
- `averageRating`, `ratingsCount`, `reviewsCount`
- `pagesCount`, `publishedYear`, `publisher`, `language`, `format`
- `genres` — genre tags
- `coverImage` — cover image URL on Goodreads' CDN

### Author (`recordType: "author"`, only when `includeMetadata: true`)

- `name`, `authorId`, `authorUrl`
- `photoUrl`
- `description` — "about the author" text
- `born`, `died` — when Goodreads publishes them
- `genres` — the author's top genres
- `website` — external author site, when listed

### Series (`recordType: "series"`, only when `includeMetadata: true`)

- `name`, `seriesId`, `seriesUrl`
- `description`
- `primaryAuthor`
- `bookCount`

### List (`recordType: "list"`, only when `includeMetadata: true`)

- `name`, `listId`, `listUrl`
- `description`
- `bookCount`, `voterCount`

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `auto` | `auto` (run every non-empty input below), `books`, `search`, `isbns`, `authors`, `series`, `lists`, `shelves`, or `genres` |
| `bookUrls` | array | `[]` | Direct Goodreads `/book/show/` URLs |
| `searchQueries` | array | `[]` | Free-text search terms (titles, authors, keywords) |
| `isbns` | array | `[]` | ISBN-10 or ISBN-13 codes |
| `authorUrls` | array | `[]` | Goodreads author URLs; every book by that author is scraped |
| `seriesUrls` | array | `[]` | Goodreads series URLs; every book in the series is scraped |
| `listUrls` | array | `[]` | Goodreads `/list/show/` Listopia URLs, paginated automatically |
| `shelfNames` | array | `[]` | Popular shelf names, e.g. `mystery`, `fantasy`, `historical-fiction` |
| `genreNames` | array | `[]` | Genre names, e.g. `fiction`, `romance`, `mystery` |
| `maxItems` | integer | `50` | Hard cap on total records emitted across all inputs |
| `maxResultsPerQuery` | integer | – | Deprecated; used as the cap only when set and `maxItems` is left at its default |
| `minRating` | integer | – | Drop books with `averageRating` below this (0-5) |
| `minRatingsCount` | integer | – | Drop books with fewer ratings than this |
| `publishYearMin` | integer | – | Drop books published before this year |
| `publishYearMax` | integer | – | Drop books published after this year |
| `language` | string | – | Keep only books whose `language` starts with this text (case-insensitive), e.g. `en` matches `en`/`eng`/`English` |
| `containsGenre` | string | – | Keep only books with a genre containing this text |
| `containsAuthor` | string | – | Keep only books with an author name containing this text |
| `includeMetadata` | boolean | `false` | Also emit an author/series/list summary record alongside the books |
| `proxyConfiguration` | object | `{"useApifyProxy": false}` | Optional proxy; Goodreads is reachable without one for most runs |

All filters apply across every mode, and only reject a record when the filtered field is present and out of range — a record missing that field passes through untouched.

### Example: book by URL plus a search query

```json
{
  "mode": "auto",
  "bookUrls": ["https://www.goodreads.com/book/show/4671.The_Great_Gatsby"],
  "searchQueries": ["sapiens"],
  "maxItems": 5
}
```

### Example: ISBN lookup

```json
{
  "mode": "isbns",
  "isbns": ["978-0-7432-7356-5", "0747532699"]
}
```

### Example: every book by an author, with a summary record

```json
{
  "mode": "authors",
  "authorUrls": ["https://www.goodreads.com/author/show/1077326.J_K_Rowling"],
  "maxItems": 20,
  "includeMetadata": true
}
```

### Example: highly-rated recent fantasy from a shelf

```json
{
  "mode": "shelves",
  "shelfNames": ["fantasy"],
  "minRating": 4,
  "minRatingsCount": 10000,
  "publishYearMin": 2020,
  "maxItems": 50
}
```

## Use cases

- Bulk-import book metadata into a library or catalog system by ISBN.
- Build a book recommendation dataset from genre, rating, and author data.
- Pull every book by an author or every entry in a series for a reading-order guide.
- Mine a Listopia list like "Best Books of the Decade" for a curated discovery feed.
- Track ratings and review counts across a shelf or genre over time.

## FAQ

**Is this affiliated with Goodreads or Amazon?**
No. This is a third-party actor that reads Goodreads' own public pages.

**Do I need a proxy or login?**
No. Goodreads' public pages work without an account, and the default run doesn't use a proxy; `proxyConfiguration` is available if you want to supply your own.

**What does `mode: "auto"` do?**
It runs every input array you've populated (`bookUrls`, `searchQueries`, `isbns`, and so on) in the same run, so you don't have to pick a single mode when you have more than one kind of input.

**Why is `includeMetadata` off by default?**
With it off, every record has `recordType: "book"`, which keeps the dataset uniform. Turning it on mixes in `author`/`series`/`list` summary records, which is useful for analysis but changes the record shape.

**Why might a `coverImage` link not load?**
The URL is whatever Goodreads publishes for that edition; very old or rare titles sometimes reference an image Goodreads itself no longer serves.

**Does `mode: "shelves"` cover an entire shelf?**
Goodreads' own shelf pages stop returning new books after roughly the first 50, regardless of how many pages you request; for deeper coverage, use `mode: "lists"` (Listopia lists paginate into the thousands) or `mode: "genres"`.

**Are individual review texts included?**
No. Book records include `reviewsCount` (the total number of reviews Goodreads reports), but the actor doesn't walk individual review pages.

**Can I combine several filters at once?**
Yes — `minRating`, `minRatingsCount`, `publishYearMin`/`publishYearMax`, `language`, `containsGenre`, and `containsAuthor` all apply together, in every mode.

**How current is the data?**
Each run reads Goodreads live, so ratings, review counts, and descriptions reflect what the site shows at run time.
