# Twitter (X) Space Scraper

Collect public X Spaces without login. Get the Space title, state, schedule, live and replay listener counts, host and participants, the replay audio link with a downloadable file, and an optional spoken-word transcript — all from public Space links, Space IDs or the post that announces a Space.

## What this actor does

- **Multiple input styles:** Space links, bare Space IDs, or an announcement post link — the actor finds the Space automatically
- **Full Space metadata:** title, state, schedule and run times, duration, listener counts, topics, host and participant profiles
- **Participants as rows:** one row per host, speaker and listener for easy filtering and export
- **Replay audio:** the streaming replay link plus a downloadable replay file
- **Transcripts:** captions when available, Whisper AI otherwise, with an optional one-row-per-segment mode
- **Empty fields are omitted**

## Output per Space

Every run returns rows with a `type`:

| type | Meaning |
|---|---|
| `space` | One Space with metadata, counts, host and participants |
| `space participant` | One row per host, speaker or listener |
| `video transcript` | The Space transcript with timed segments |
| `space transcript segment` | One row per transcript segment (when segment rows are enabled) |
| `status` | A problem row (invalid link, deleted Space, rate limit, unavailable replay); always contains a `message` |

### Space row fields

- `id`, `url`, `title`, `state` (`Live`, `Ended` or `Scheduled`)
- `createdAt`, `scheduledStart`, `startedAt`, `endedAt`, `durationSeconds`, `updatedAt`
- `listenersLive`, `listenersReplay`, `replayAvailable`
- `mediaKey`, `topics[]`, `isMuted`
- `creator` — full profile object (username, name, bio, followers, profile image, verification)
- `participants[]` — each with `role` (`admin`, `speaker` or `listener`) and the full `user` profile
- `sourceTweetId`, `sourceTweetUrl` — when you pasted the announcement post
- `replayAudioUrl` — the streaming replay link (when replay audio is requested)
- `replayAudioFileUrl`, `replayAudioFileBytes`, `replayAudioContentType` — the downloadable replay file (when one can be produced)
- `collectedAt`

### Participant row fields

- `spaceId`, `spaceUrl`, `spaceTitle`, `participantIndex`
- `role`, `userId`, `username`, `name`, `bio`
- `followers`, `following`, `profileImageUrl`, `isBlueVerified`, `verified`
- `collectedAt`

### Transcript row fields

- `spaceId`, `spaceUrl`, `title`
- `language`, `transcriptionMethod` (`captions` or `ai`), `modelUsed`
- `segmentCount`, `fullTranscript`, `segments[]` (`index`, `start`, `end`, `text`)
- `collectedAt`

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `spaceUrls` | array | – | Space links, Space IDs, or an announcement post link (at least one required) |
| `includeParticipants` | boolean | `true` | Host and participants on the Space row + one participant row each |
| `includeReplayAudio` | boolean | `false` | Streaming replay link + downloadable replay file |
| `includeTranscript` | boolean | `false` | Produce a transcript of the replay audio |
| `transcriptSegmentsAsRows` | boolean | `false` | Also emit one row per transcript segment |
| `transcriptionMethod` | string | `auto` | `auto` (captions first, then AI), `captions` (fastest), `ai` (Whisper) |
| `whisperModel` | string | `base` | `tiny`, `base`, `small`, `medium`, `large-v3` |
| `proxyConfiguration` | object | – | Optional, for large or repeated runs |

### Example: metadata and participants

```json
{
  "spaceUrls": ["https://x.com/i/spaces/1YpKkZgnrDXxj"],
  "includeParticipants": true
}
```

### Example: replay audio file

```json
{
  "spaceUrls": ["1YpKkZgnrDXxj", "https://x.com/i/spaces/1YpKkZgnrDXxj"],
  "includeReplayAudio": true
}
```

### Example: transcript with segment rows

```json
{
  "spaceUrls": ["https://x.com/i/spaces/1YpKkZgnrDXxj"],
  "includeTranscript": true,
  "transcriptionMethod": "auto",
  "transcriptSegmentsAsRows": true
}
```

### Example record

```json
{
  "type": "space",
  "id": "1YpKkZgnrDXxj",
  "url": "https://x.com/i/spaces/1YpKkZgnrDXxj",
  "title": "Meet NASA's New Mission Control Flight Directors #AskNASA",
  "state": "Ended",
  "createdAt": "2022-07-19T15:25:30+00:00",
  "startedAt": "2022-07-19T15:59:30+00:00",
  "endedAt": "2022-07-19T16:45:16+00:00",
  "durationSeconds": 2746.0,
  "listenersLive": 47951,
  "listenersReplay": 112570,
  "replayAvailable": true,
  "mediaKey": "28_1549415139677327362",
  "topics": ["Space"],
  "creator": { "username": "NASA", "name": "NASA" },
  "participants": [{ "role": "speaker", "username": "Space_Station" }],
  "collectedAt": "2026-10-08T09:15:00+00:00"
}
```

## Use cases

- **Community research** — capture speakers, topics and audience size for public Spaces
- **Media monitoring** — archive broadcasts relevant to your industry as soon as they end
- **Transcripts** — turn long-form audio into searchable, timestamped text
- **Audience analytics** — compare live vs replay listeners across Spaces
- **Creator tooling** — pull participant lists and replay files for recaps and show notes
- **Compliance and archiving** — keep replay audio and metadata for public events

## FAQ

**Do I need a Twitter/X account or cookies?** No. The actor is fully cookieless and reads only public Spaces.

**Can it collect a Space from an announcement post?** Yes. Paste the post link and the actor finds the Space inside it automatically. If the post does not contain a Space, you get a clear `status` row.

**Why is replay audio sometimes unavailable?** Live Spaces have no replay yet, and some Spaces are not kept available for replay. In both cases you get an honest `status` row explaining why.

**How is the replay file produced?** The public replay stream is downloaded and combined into one playable audio file, then stored with a download link. Very large replays (above roughly 250 MB) may exceed the file cap — you still get the streaming replay link.

**How do transcripts work?** With `auto` the actor uses captions when the replay has them and otherwise runs Whisper AI over the audio. Captions are fast; AI takes time proportional to the Space length. `transcriptionMethod` lets you force one or the other.

**Which Whisper model should I use?** `base` is a good balance. Larger models are more accurate but slower. Long Spaces are processed in 10-minute chunks, and the actor automatically falls back to a smaller model if the run does not have enough memory.

**How many rows will a full Space run produce?** A Space with participants and a full transcript can produce several hundred rows — longer Spaces with more participants and transcript segments produce proportionally more. There is no artificial row cap; the row count follows the Space's real length and participant count.

**Does it support private or deleted Spaces?** No. Private, deleted or never-existing Spaces return a `status` row.

**Is this affiliated with X?** No. This is a third-party actor that reads publicly visible content.

## Limits

- Public Spaces only; private, deleted or unavailable Spaces produce a `status` row.
- AI transcription can take several minutes for long Spaces; captions are near-instant when available.
- Live Spaces have no replay yet, so replay audio and transcript options require a finished Space.

## Integrations

Run it on a schedule and send Space summaries, participant lists or transcripts to Google Sheets, Airtable, Slack or any webhook through Apify's built-in integrations, or call the actor from the Apify API.
