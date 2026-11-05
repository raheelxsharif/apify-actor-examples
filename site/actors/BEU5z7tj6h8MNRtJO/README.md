# YouTube SponsorBlock Segments Scraper

Get community-reported [SponsorBlock](https://sponsor.ajay.app/) segments for any public YouTube video: sponsor reads, intros, outros, self-promotions, filler tangents, highlights, and more — with exact start/end times, category, action type, and community votes.

No cookies, no YouTube login, and no video download. The actor reads the public SponsorBlock database directly.

## What this actor does

- **Video lookup:** pass any YouTube video URL or ID
- **All segment categories:** sponsor, intro, outro, self-promo, interaction, music, preview, and more
- **Timing data:** start, end, and duration for every segment
- **Bulk input:** multiple videos per run
- **Empty fields are omitted** — records only contain values that are actually populated.

## Output


One dataset row per segment, plus a truthful status row when a video has no segments or the input is invalid.

| Field | Example | Description |
|---|---|---|
| videoId | `"aQp-iM0bCs4"` | YouTube video ID |
| videoUrl | `"https://www.youtube.com/watch?v=aQp-iM0bCs4"` | Canonical watch URL |
| segmentId | `"b772d4b7..."` | SponsorBlock segment UUID |
| category | `"sponsor"` | SponsorBlock category key |
| categoryName | `"Sponsor"` | Human-readable category |
| actionType | `"skip"` | Skip, mute, full video, or highlight/POI |
| startTime | `0` | Segment start in seconds |
| endTime | `31.861` | Segment end in seconds |
| durationSeconds | `31.861` | Segment length in seconds |
| videoDurationSeconds | `361` | Full video length in seconds |
| votes | `0` | Community votes for the segment |
| locked | `false` | Whether the segment is locked from edits |
| description | `"..."` | Optional community note |
| scrapedAt | `"2026-10-02T..."` | Capture timestamp |

Status rows:

| status | Meaning |
|---|---|
| `no_segments` | The video is public and readable but has no SponsorBlock segments. |
| `invalid_video` | The input was not a YouTube video URL or ID. |
| `failed` | SponsorBlock did not return a usable response after retries. |

Status rows carry the same `videoId`, `videoUrl`, and `scrapedAt` fields as segment rows, plus an `error` field with a short explanation when `status` is `invalid_video` or `failed`.

Every field is omitted when YouTube/SponsorBlock does not provide a value — there are no null placeholders.

## Input


| Field | Type | Default | Description |
|---|---|---|---|
| `videoUrls` | array | — (required) | YouTube video URLs or 11-character video IDs. Watch, `youtu.be`, Shorts, live, and embed links are accepted. |
| `categories` | array (dropdown) | all | Filter to specific SponsorBlock categories: Sponsor, Unpaid/Self Promotion, Interaction Reminder, Intro, Outro, Preview/Recap, Non-Music Section, Filler Tangent, Highlight, Exclusive Access, Chapter. |
| `actionTypes` | array (dropdown) | all | Filter to segments whose action is Skip, Mute, Full video, or Highlight/POI. |
| `minVotes` | integer | `0` | Drop segments with fewer community votes than this. `0` keeps every segment; higher values keep only well-confirmed segments. |
| `includeDescriptions` | boolean | `true` | Include the community-written note attached to a segment when one exists. |

### Example input

```json
{
    "videoUrls": ["https://www.youtube.com/watch?v=aQp-iM0bCs4"],
    "categories": ["sponsor", "intro"],
    "minVotes": 0,
    "includeDescriptions": true
}
```

## Examples

### Example: one video

```json
{
  "videoUrls": ["https://www.youtube.com/watch?v=dQw4w9WgXcQ"]
}
```

### Example: multiple videos with all categories

```json
{
  "videoUrls": [
    "https://www.youtube.com/watch?v=dQw4w9WgXcQ",
    "https://www.youtube.com/watch?v=9bZkp7q19f0"
  ]
}
```

## Use cases


- **Skip sponsor reads automatically** — build player overlays or cut lists from exact segment times.
- **Transcript cleanup** — remove sponsor/intro/outro text before analysis; pairs with the YouTube Transcript Scraper.
- **Sponsor-load analytics** — measure how much of a channel's runtime is sponsored content.
- **Dataset enrichment** — join segments onto video metadata by `videoId`.

## FAQ

**What is SponsorBlock?**

SponsorBlock is a public crowdsourced database of timestamped segments in YouTube videos (sponsors, intros, outros, and more).

**Do I need an API key?**

No. The actor reads SponsorBlock's public API.

**What happens when a video has no segments?**

The row is returned with an empty segment list omitted; a status message explains that no segments exist for the video.

**How fresh is the data?**

Segments are community-submitted and reflect the SponsorBlock database at run time.

**Is this affiliated with YouTube or SponsorBlock?**

No. This actor reads the public SponsorBlock API; it is not affiliated with, endorsed by, or sponsored by YouTube or the SponsorBlock project.

**Why does `votes` sometimes show `0` for a segment that still appears valid?**

SponsorBlock segments go live as soon as they're submitted and don't require positive votes to be returned by the API — a `0` simply means the segment hasn't yet accumulated community votes, not that it was rejected.
