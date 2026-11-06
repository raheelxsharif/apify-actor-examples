# Snapchat Highlight & Story Media Downloader

Collect the individual snaps inside public Snapchat highlight albums and active stories, then download them from the direct media URLs in each record. Add creator profile details, captions, timestamps and parent-story metadata without a Snapchat login, cookies or an API key. Point it at usernames or paste highlight/story links — one row per snap.

## What this actor does

- **Two modes:** `profiles` for a list of usernames/profile URLs, and `urls` for pasted profile, highlight or active-story snap links.
- **Both story surfaces:** saved curated highlights and the creator's current public story, with a `storySource` value on every row.
- **Download-ready media:** direct `mediaUrl` (MP4 or JPEG), `mediaPreviewUrl`, plus optional `overlayUrl` and `attachmentUrl`.
- **Durable copies:** `downloadMedia` downloads each snap file and preview, plus the avatar, hero image, snapcode, cover, overlay and transcription into the run's Apify key-value store, returns a permanent public URL and keeps the original CDN link in a `*Original` field (up to 200 files and 50 MB per file per run).
- **Creator context:** display name, verification, subscribers, bio, avatar, website, category, address, hero image and profile timestamps repeat on every snap.
- **Incremental runs:** `postedAfter` skips long-lived highlight snaps you already collected, and `maxItems`, `maxSnapsPerUser` and `maxHighlightsPerUser` keep every run bounded.
- **Clean rows:** empty fields are omitted — a snap without geotags, captions or a sponsorship flag simply has no such key.

## Output per snap

- `recordId` — stable natural key: the snap ID for active stories, or highlight ID + snap position for saved highlights
- `recordType`, `targetType`, `profileStatus` — `snap` / `profile`, `highlightUrl` or `storySnapUrl`; public/private/notFound/noContent/unavailable
- `username`, `displayName`, `isVerified`, `profileUrl` — creator identity
- `subscriberCount`, `bio`, `profilePictureUrl` — public profile details
- `websiteUrl`, `address`, `profileCategory`, `profileSubcategory` — public profile extras when present
- `snapcodeImageUrl`, `profileHeroImageUrl` — Snapcode and profile hero image
- `profileCreatedAt`, `profileUpdatedAt` — profile history timestamps
- `storySource` — `curatedHighlight` or `activeStory`
- `storyId`, `storyType`, `highlightId`, `storyTitle`, `highlightThumbnailUrl` — parent story identity and cover
- `highlightIndex`, `highlightSnapCount` — position and size of the parent highlight
- `snapIndex`, `snapId`, `snapMediaType`, `mediaType` — position, raw type and normalised `image`/`video`
- `mediaUrl`, `mediaPreviewUrl`, `overlayUrl`, `attachmentUrl` — direct download and preview files
- `postedAt`, `intervalStartTimeMs` — post time and any interval start supplied by Snapchat
- `snapTitle`, `snapSubtitles`, `lat`, `lng`, `hasAttachment`, `isSponsored` — per-snap details when present
- `audioTranscriptionObjectUrl` — transcription file link when Snapchat supplies one
- `storySubtitle`, `storyShareId`, `emoji`, `storyTapId`, `canonicalUrlSuffix`, `videoTrackUrl`, `isAttributed`, `thumbnailTiles` — parent-story extras
- `mediaContentType`, `mediaSizeBytes` — MIME type and byte size when `downloadMedia` is on
- `mediaUrlOriginal`, `mediaPreviewUrlOriginal`, `overlayUrlOriginal`, `attachmentUrlOriginal`, `profilePictureUrlOriginal`, `profileHeroImageUrlOriginal`, `snapcodeImageUrlOriginal`, `highlightThumbnailUrlOriginal`, `videoTrackUrlOriginal`, `audioTranscriptionObjectUrlOriginal` — original CDN URLs kept when a field is rehosted
- `sourceUrl` — the profile, highlight or story-snap page this row came from
- `scrapedAt` — when the record was fetched
- `inputUsername`, `error` — present instead of the snap fields on a failed target

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `profiles` | `profiles` or `urls`. |
| `usernames` | array | – | Usernames or profile URLs (mode=profiles). Bare, `@name`, `/add/name` and `/@name` forms work. |
| `startUrls` | array | – | Profile, highlight or story-snap links (mode=urls). |
| `sources` | array | `["curatedHighlights", "activeStory"]` | `curatedHighlights` and/or `activeStory`. |
| `maxSnapsPerUser` | int | `50` | Maximum snaps per target (1–10,000). |
| `maxHighlightsPerUser` | int | `0` | How many highlights to read per profile; `0` means all rendered highlights (max 100). |
| `maxItems` | int | `5000` | Hard cap on records across the whole run (1–100,000). |
| `postedAfter` | string | – | Keep snaps posted on or after this date (`YYYY-MM-DD` or ISO 8601). |
| `downloadMedia` | boolean | `false` | Download media into the run's key-value store and return permanent public URLs. Capped at 200 files and 50 MB per file per run; failures fall back to the direct URL. |
| `proxyConfiguration` | object | – | Optional Apify proxy override; public profiles need no proxy. |

### Example: every snap from a creator

```json
{
  "mode": "profiles",
  "usernames": ["brentrivera", "nba"],
  "sources": ["curatedHighlights", "activeStory"],
  "maxSnapsPerUser": 300,
  "maxItems": 600
}
```

### Example: paste a shared highlight link

```json
{
  "mode": "urls",
  "startUrls": [
    { "url": "https://www.snapchat.com/@kyliejenner/highlight/367bdbd7-d19a-4bf1-bc81-c3e73f7b1809" }
  ],
  "maxSnapsPerUser": 100
}
```

### Example: only new snaps since the last run

```json
{
  "mode": "profiles",
  "usernames": ["khaby.lame"],
  "sources": ["curatedHighlights"],
  "postedAfter": "2026-10-01",
  "maxItems": 100
}
```

### Example: keep a durable copy of each file

```json
{
  "mode": "profiles",
  "usernames": ["fcbarcelona"],
  "sources": ["activeStory"],
  "maxSnapsPerUser": 20,
  "downloadMedia": true
}
```

## Use cases

- **Content archiving** — save a permanent copy of a creator's highlight reel before it is edited or removed.
- **Influencer monitoring** — track what a creator posts across saved highlights and the live story in one dataset.
- **Campaign research** — pull sponsored snaps across several creators with `isSponsored` intact.
- **Social-listening pipelines** — feed structured snap metadata, captions and timestamps into analytics.
- **Scheduled collection** — use `postedAfter` and `maxItems` to run daily or weekly without re-downloading the same old highlights.

## FAQ

**Do I need a Snapchat account or cookies?**
No. The actor reads public profile and highlight pages over plain HTTP.

**What is the difference between a curated highlight and an active story?**
Highlights are albums a creator saves and keeps on their profile. The active story is the current public 24-hour story. Both are collected by default; `sources` limits the run to one.

**How many highlights can a profile have?**
Snapchat renders up to 18 saved highlights on the public profile page, and that is the ceiling this actor can see from a username. If you have a direct link to another highlight, paste it in `urls` mode to retrieve its snaps.

**Why did a pasted story link return an error?**
Active stories are ephemeral. If the snap has expired, Snapchat no longer serves it and the actor records a `profileStatus: "noContent"` error row instead of failing the run.

**Will `mediaUrl` still work later?**
Snapchat CDN links carry rotating signature tokens. They work immediately and usually for some time, but they are not permanent. Set `downloadMedia` to true if you need links that keep working. Rehosting is limited to 200 files and 50 MB per file per run; anything over a limit keeps the direct CDN URL.

**How does `maxSnapsPerUser` interact with `maxItems`?**
`maxSnapsPerUser` caps each profile, highlight or story-snap target. `maxItems` caps the whole run across every target. The run stops cleanly when either limit is reached.

**What happens to private or unknown accounts?**
The actor writes one diagnostic row per failed target with `inputUsername`, `error` and `profileStatus`, then continues with the remaining targets.

**Are duplicate snaps possible?**
Each row has a deterministic `recordId`. Repeated targets and any record whose `recordId` was already emitted in the run are dropped, so overlapping inputs do not create duplicate rows.

**Does this actor collect Spotlight, comments or lenses?**
No. Those are separate Snapchat surfaces with their own actors.

**Is this affiliated with Snapchat?**
No. It is an independent third-party actor that reads public Snapchat pages and is not operated by or affiliated with Snap Inc.
