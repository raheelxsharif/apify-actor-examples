# X Space Collector — Metadata, Hosts, Replay & Transcripts

Collect public X (Twitter) Spaces without an account, cookies or API keys. Start from Space links, IDs or announcement posts — or let the actor discover Spaces from a keyword, hashtag or host handle. Every run returns clean, omit-empty records for the Space, its people, its replay audio and, when requested, a timed transcript.

## Three ways to find Spaces

| Mode | Input | What happens |
|---|---|---|
| Exact Spaces | `spaceUrls` | Reads a Space link, a bare Space ID, or the post that announces a Space. The Space card is resolved automatically. |
| Keyword / hashtag | `searchQueries` | Reads recent public posts matching the query and resolves the Spaces they announce. |
| Host | `hostUsernames` | Reads a host's recent public posts (and their search results) and resolves the Spaces they announce. |

All three modes can run in one go. Discovered Spaces are capped by `maxSpaces`; pasted links and IDs are always collected. When several inputs resolve to the same Space (for example its link and its announcement post), it is collected once. `spaceState` can keep only live, scheduled, ended or any Spaces.

## What you get

Each run writes rows with a `type`:

| type | Meaning |
|---|---|
| `space` | One Space with metadata, counts, host and participants |
| `space participant` | One row per host, speaker or listener |
| `video transcript` | The transcript summary with timed segments |
| `space transcript segment` | One row per timed segment (optional) |
| `status` | An honest problem row (invalid link, deleted Space, rate limit, unavailable replay) |

### Space row

- **Identity:** `id`, `url`, `title`, `state`, `contentType`
- **Times:** `createdAt`, `scheduledStart`, `startedAt`, `endedAt`, `durationSeconds`, `updatedAt`, `replayStartSeconds`
- **Audience:** `listenersLive`, `listenersReplay`, `replayAvailable`
- **People:** `creator` (full public profile), `hostId`, `hostUsername`, `hostName`, `participants[]`, `participantCounts`
- **Space settings:** `topics`, `topicIds`, `mentionedUserIds`, `isMuted`, `isLocked`, `isEmployeeOnly`, `disallowJoin`, `noIncognito`, `conversationControls`, `maxAdminCapacity`, `maxGuestSessions`, `narrowCastSpaceType`
- **Replay:** `replayAudioUrl`, `replayAudioFileUrl`, `replayAudioFileBytes`, `replayAudioContentType`, `replayAudioDurationSeconds`
- **Source:** `sourceTweetId`, `sourceTweetUrl` when you started from an announcement post
- `mediaKey`, `collectedAt`

### Participant row

`spaceId`, `spaceUrl`, `spaceTitle`, `participantIndex`, `role`, `userId`, `username`, `name`, `bio`, `location`, `website`, `followers`, `following`, `postCount`, `profileImageUrl`, `bannerUrl`, `isBlueVerified`, `verified`, `verifiedType`, `periscopeUserId`, `isMutedByAdmin`, `isMutedByGuest`, `collectedAt`.

### Transcript row

`spaceId`, `spaceUrl`, `title`, `language`, `transcriptionMethod` (`captions` or `ai`), `modelUsed`, `segmentCount`, `fullTranscript`, `segments[]` (`index`, `start`, `end`, `text`), `collectedAt`.

Empty fields are omitted at every depth. There are no `null` values in the dataset.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `spaceUrls` | array | – | Space links, Space IDs, or announcement post links |
| `searchQueries` | array | – | Keywords or hashtags for Space discovery |
| `hostUsernames` | array | – | Handles whose recent posts announce Spaces |
| `maxSpaces` | integer | `20` | Cap for discovered Spaces (1–200) |
| `spaceState` | string | `any` | `any`, `live`, `upcoming` or `ended` |
| `includeParticipants` | boolean | `true` | Participant list on the Space row + participant rows |
| `includeParticipantProfiles` | boolean | `true` | Enrich participant stubs with public profile details |
| `maxParticipantProfiles` | integer | `50` | Profile lookups per Space (0 = all) |
| `includeReplayAudio` | boolean | `false` | Replay HLS link + downloadable audio file |
| `includeTranscript` | boolean | `false` | Produce a transcript from captions or Whisper |
| `transcriptSegmentsAsRows` | boolean | `false` | Emit one row per transcript segment |
| `transcriptionMethod` | string | `auto` | `auto`, `captions` or `ai` |
| `whisperModel` | string | `base` | `tiny`, `base`, `small`, `medium` or `large-v3` |
| `proxyConfiguration` | object | – | Optional; automatic fallback is built in |

At least one of `spaceUrls`, `searchQueries` or `hostUsernames` is needed.

### Examples

Metadata and participants:

```json
{
  "spaceUrls": ["https://x.com/i/spaces/1YpKkZgnrDXxj"],
  "includeParticipants": true
}
```

Discover by keyword, keep only ended Spaces:

```json
{
  "searchQueries": ["#Space", "NASA"],
  "spaceState": "ended",
  "maxSpaces": 10
}
```

Discover from hosts:

```json
{
  "hostUsernames": ["NASA", "@SpaceX"],
  "maxSpaces": 5
}
```

Replay audio with a transcript:

```json
{
  "spaceUrls": ["1YpKkZgnrDXxj"],
  "includeReplayAudio": true,
  "includeTranscript": true,
  "transcriptionMethod": "auto",
  "transcriptSegmentsAsRows": true
}
```

### Example Space record

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
  "hostUsername": "NASA",
  "participantCounts": {"total": 10, "admin": 1, "speaker": 9},
  "topics": ["NASA", "Science", "Space"],
  "collectedAt": "2026-11-07T09:15:00+00:00"
}
```

## How discovery works

Spaces are discovered from public posts, not from a private search index. The actor reads recent matching posts through X's public guest surfaces and resolves each post's Space card. This keeps the actor cookieless. Consequences:

- Discovery looks at recent public posts, not the entire archive.
- A Space with no public announcement post cannot be discovered by keyword or host. Paste its link or ID instead.
- Guest search is the least reliable public surface; a query can return fewer posts at busy times. The actor retries and falls back to the host's own timeline.
- X's signed-in Space search and topic browser are not available to guest callers. They are deliberately not faked.

## FAQ

**Do I need an X account, cookies or an API key?** No. The actor is fully cookieless and reads only public Spaces, public posts and public profiles.

**Does it work with private or deleted Spaces?** No. They produce a clear `status` row.

**Why is replay audio sometimes unavailable?** Live Spaces have no replay yet, and some hosts disable replay. You get a `status` row explaining which case applies.

**How is the replay file produced?** The public replay stream is downloaded and combined into one playable audio file, then stored with a stable download link. Very large replays (above roughly 250 MB) may exceed the file cap — you still get the streaming replay link.

**How do transcripts work?** With `auto`, native captions are used when the replay has them; otherwise Whisper AI runs over the audio. Captions are fast. AI time grows with the Space length, and long Spaces are split into 10-minute chunks to bound memory.

**Which Whisper model should I use?** `base` is the best balance on the default 1 GB. Larger models need more actor memory. The actor downgrades safely instead of crashing, and reports the model it actually used in `modelUsed`.

**How many rows will a run produce?** A Space with participants and a full transcript can produce several hundred rows. Longer Spaces and larger participant lists produce more. Discovery is capped by `maxSpaces`; pasted Spaces are not capped.

**Can I schedule it?** Yes. Use `searchQueries` or `hostUsernames` with a schedule to pick up newly announced Spaces, or run it after a recurring show with the host's handle.

**Is this affiliated with X?** No. It is a third-party actor that reads publicly visible content.

## Limits

- Public Spaces only; private, deleted or unavailable Spaces return a `status` row.
- Keyword/hashtag/host discovery covers recent public posts, not the full archive.
- Signed-in Space search and the topic browser are not reachable without credentials and are not supported.
- Replay audio and transcripts require a finished Space with replay enabled.
- AI transcription is CPU-bound; on 1 GB the actor runs `tiny`/`base` and may downgrade a larger requested model.
- Participant profile enrichment stops at `maxParticipantProfiles` per Space.

## Integrations

Use Apify's built-in integrations to send Space summaries, participant lists or transcripts to Google Sheets, Airtable, Slack, a webhook or your own API.
