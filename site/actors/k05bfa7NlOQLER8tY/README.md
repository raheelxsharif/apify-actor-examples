# TikTok Keyword & Brand Mention Monitor

Find the TikTok posts that actually mention your brand, product, campaign, hashtag or handle — and see why each one matched. This actor reads TikTok's public search from several angles (keyword, hashtag, handle, photo posts, related searches and optional brand sounds), merges duplicates into one row per post, and labels the evidence so a real caption mention is never confused with a weak search hit.

## What this actor does

- **Six source types:** keyword search, `#hashtag` search, `@handle` search, photo/slideshow search, token-filtered related searches, and explicit brand/campaign sounds.
- **Evidence, not guesses:** every row says whether the term appears in the caption, in a hashtag, as TikTok's clickable `@mention`, from a sound page, or only as a search result.
- **Strict by default:** `mentionsOnly` keeps rows with real evidence; switch to `allResults` to keep every search hit, labelled.
- **One row per post:** multi-keyword and multi-source duplicates are merged, with `matchedKeywords[]`, `sources[]` and the best `rank` across merged sources.
- **Built for monitoring:** server-backed date windows, newest-first ordering, an exact cut-off, and optional only-new delta state.
- **Full post detail:** caption, engagement, creator snapshot, sound, video or slideshow images, hashtags, mentions and location tags.
- **Honest empty fields:** values TikTok does not provide are omitted, never filled with placeholders.

## Output per post

Every row is one public TikTok post.

- `postId`, `postType` (`video` or `slideshow`), `postUrl` — identity and canonical link
- `caption`, `textLanguage`, `createTime`, `createdAt` — text and publication time
- `keyword` — the monitored term that primarily matched this post; an explicit sound-only row uses `sound:<musicId>`
- `source` — the primary source: `keyword`, `hashtag`, `handle`, `photo`, `relatedSearch` or `sound`; merged rows list every source in `sources[]`
- `matchReason` — `caption`, `hashtag`, `taggedMention`, `sound` or `searchResult`
- `matchReasons[]` — every reason this post matched across sources
- `matchedTerm`, `matchedKeywords[]`, `sources[]` — full provenance after dedupe; explicit sounds contribute a `sound:<musicId>` label
- `mentionInCaption`, `captionMatch`, `hashtagMatch`, `taggedMatch`, `soundMatch` — evidence flags
- `matchedHashtags[]` — the hashtag entries that matched the monitored term
- `matchedMention` — `{username, displayText, userId, secUid, start, end}` for a clickable tag
- `rank` — best (lowest) position this post had across all merged sources
- `searchUrl`, `searchCountry`, `monitorName`, `scrapedAt` — provenance and run context
- `isNew`, `firstSeenAt` — monitor state (first run: everything is new)
- `playCount`, `likeCount`, `commentCount`, `shareCount`, `collectCount`, `repostCount`
- `author` — `{id, secUid, username, displayName, bio, verified, privateAccount, avatarUrl, followerCount, followingCount, heartCount, videoCount}`
- `music` — `{id, title, authorName, original, duration, album, playUrl, coverUrl, ...}` when a sound is attached
- `video` — dimensions, duration, definition, `playUrl`, `downloadAddr`, cover images and quality details
- `images[]` — ordered slideshow images (`{ordinal, width, height, urlList}`)
- `hashtags[]` — all structured hashtags in the post
- `mentions[]` — all clickable `@mentions`, with real handles and source-text offsets
- `poi` — location tag, when the creator added one
- `isAd`, `adLabelVersion`, `privateItem`, `forFriend`, `originalItem`, `stitchEnabled`, `shareEnabled`, `duetEnabled`, `commentStatus`
- `isAigc`, `aiContentLabel`, `isPaidPartnership` — only when TikTok exposes a verified label
- `mediaUrlExpiresAt` — when TikTok's signed media links expire

Media URLs (`video.playUrl`, `video.downloadAddr`, `music.playUrl`) are reference-only. Most video play/download links return 403 outside TikTok; music and cover links are signed too and can expire or fail. Covers and slideshow images also expire.

## How matching works

A search result is not automatically a mention.

- `caption` — the monitored term appears contiguously in the caption. Single words use whole-word matching, so `nike` does **not** match `nikeshoes`. Multi-word terms must appear as a true phrase: `#nike ... #shoes` is **not** a `nike shoes` caption match; `my nike shoes` is. For non-Latin scripts, matching is a contiguous substring, so related forms can count (for example `القهوة` for `قهوة`).
- `hashtag` — the post carries a hashtag whose slug equals the monitored term with whitespace removed. `nike shoes` matches `#nikeshoes`; separate `#nike` and `#shoes` do not count as a phrase match.
- `taggedMention` — the caption contains TikTok's clickable tag for the monitored `@handle`. A direct `@handle` keyword is strict: in `mentionsOnly`, only clickable tags (or an explicit sound source) are kept; caption and hashtag hits for the bare name are labelled `searchResult` and appear only in `allResults`.
- `sound` — the post came from a sound URL you supplied.
- `searchResult` — TikTok returned the post for the search, but none of the above is true. These rows are kept only in `allResults` mode.

For non-Latin scripts (for example Arabic, Chinese or Devanagari), TikTok often returns relevant posts whose captions do not contain the literal query as a separate word. Matching is a contiguous substring for those scripts, so `mentionsOnly` can keep related word forms; use `allResults` when you want every search hit and label the weak ones yourself.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `keywords` | array | – | **Required.** Keywords, phrases, `#hashtags` or `@handles` to monitor. |
| `matchMode` | string | `mentionsOnly` | `mentionsOnly` (real evidence) or `allResults` (every search hit, labelled). |
| `datePosted` | string | `anyTime` | `anyTime`, `past24Hours`, `pastWeek`, `pastMonth`, `past3Months` or `past6Months`. |
| `postedAfter` | string | – | Exact cut-off: `2026-10-01` or a relative age such as `3 days`. |
| `sortBy` | string | `relevance` | `relevance`, `newest` or `mostLiked`. |
| `searchDepth` | string | `standard` | `keywordOnly`; `standard` adds #hashtag and @handle for single-word terms; `deep` adds photos, related searches and case/spacing variants. |
| `maxResultsPerKeyword` | integer | 50 | Rows kept per keyword after merging sources (1-500). |
| `maxItems` | integer | – | Optional global cap across all keywords. |
| `minPlayCount` | integer | – | Drop posts with fewer views. |
| `searchCountry` | string | `any` | `any` or a market code such as `US`, `GB`, `PK`, `JP`. |
| `onlyNewMentions` | boolean | false | Emit only posts not seen by this monitor before. |
| `monitorName` | string | `default` | State name for this watchlist. |
| `soundUrls` | array | – | Optional TikTok sound URLs or IDs to read as explicit sources. |

### Example: brand monitor, last week, newest first

```json
{
  "keywords": ["nike", "adidas"],
  "matchMode": "mentionsOnly",
  "searchDepth": "standard",
  "datePosted": "pastWeek",
  "sortBy": "newest",
  "maxResultsPerKeyword": 100
}
```

### Example: campaign hashtag with photo posts

```json
{
  "keywords": ["#JustDoIt"],
  "matchMode": "allResults",
  "searchDepth": "deep",
  "maxResultsPerKeyword": 200,
  "minPlayCount": 1000
}
```

### Example: unicode discovery

```json
{
  "keywords": ["قهوة", "咖啡"],
  "matchMode": "allResults",
  "searchDepth": "standard",
  "maxResultsPerKeyword": 50
}
```

### Example: daily only-new watchlist

```json
{
  "keywords": ["khaadi", "@gymshark"],
  "matchMode": "mentionsOnly",
  "datePosted": "pastWeek",
  "sortBy": "newest",
  "onlyNewMentions": true,
  "monitorName": "brand-watch"
}
```

### Example: brand-owned sound

```json
{
  "keywords": ["khaadi"],
  "soundUrls": ["https://www.tiktok.com/music/original-sound-7627060748200577810"],
  "searchDepth": "standard",
  "maxResultsPerKeyword": 100
}
```

## Use cases

- **Brand monitoring** — catch posts that name your brand, not only posts that tag your account.
- **Competitor tracking** — compare how often competing brands are mentioned and how those posts perform.
- **Campaign measurement** — follow a campaign hashtag, slogan or brand-owned sound across creators.
- **PR and crisis watch** — run a daily only-new monitor for a product or brand term.
- **Market research** — compare the conversation in different countries and languages.
- **Creator and partnership discovery** — find the creators and posts driving a brand conversation.

## FAQ

### Do I need a TikTok account, cookies or a proxy?

No. The actor reads public TikTok search and needs no login, cookies or proxy setup. It chooses a working connection automatically.

### What is the difference between `caption`, `hashtag`, `taggedMention` and `searchResult`?

`caption` means the term appears in the caption text. `hashtag` means a matching hashtag is attached. `taggedMention` means TikTok's clickable tag for the monitored handle is present. `searchResult` means TikTok returned the post for the search but none of the stronger evidence exists; those rows appear only in `allResults` mode.

### Why do I get fewer rows than `maxResultsPerKeyword`?

TikTok's public search pool is finite and rotates. Runs returned roughly 130-240 posts per source query in current tests; a repeat of the same query can return a different set. Niche terms, narrow date windows and hashtags can run out earlier. Date windows also merge a bounded unfiltered pass, so recent posts are not lost just because TikTok's date-ranked pool is incomplete. `deep` mode searches several sources and merges them, which usually finds more.

### How does `onlyNewMentions` work?

The actor saves the posts it has seen for each `monitorName` in a persistent store. On the next run, only posts not seen before are emitted and marked `isNew`. The first run backfills everything it finds within your filters and caps; if the state is lost, the next run backfills again. "New" means new to this monitor, not new on TikTok.

### Does it monitor hashtags and handles?

Yes. A plain single-word keyword can automatically include its `#hashtag` and `@handle` in `standard` or `deep` mode. You can also enter a hashtag or handle directly. A direct `@handle` is strict in `mentionsOnly`: only TikTok's clickable tag for that handle is kept. A direct `#hashtag` keeps matching hashtag, caption or tag evidence for the bare term. Use a plain keyword when you want caption mentions of a brand name.

### Can I search a specific country?

Yes. Set `searchCountry` to a market code. Country-specific results are slower because the actor uses a country-pinned connection. Leave it on `any` for the fastest run.

### Why are some fields missing from a row?

Empty values are omitted. TikTok does not expose every field for every post. A per-post country is not available from search; `poi` appears when the creator added a location tag, and `textLanguage` gives the caption language.

### Can I download the video or sound from the output?

Treat media URLs as reference-only. Most video play/download links return 403 outside TikTok; music and cover links are signed too and can expire or fail. Covers and slideshow images also expire.

### Does this actor include comments?

No. It monitors public posts and captions. TikTok's public web search does not offer a comment search, so comment-level mentions are out of scope.

### Is this affiliated with TikTok?

No. This is a third-party actor that reads publicly visible TikTok search results and is not affiliated with TikTok or ByteDance.
