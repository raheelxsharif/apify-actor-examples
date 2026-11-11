# YouTube Live Stream & Replay Chat Extractor

Read the chat behind a YouTube live stream — while it is running or from the saved replay after it ends. Give the actor a video link, a video ID or a channel, and it returns the chat timeline as structured records: text messages, Super Chats, Super Stickers, memberships, gift memberships and paid gift messages.

No cookies, login, API key or proxy setup is required.

## What you get

- **Replay chat in full** — pages through the saved chat to the end of the stream instead of stopping after a fixed number of pages.
- **Live chat capture** — opens the live chat feed, emits the recent history already on screen, then polls at YouTube's own interval until your time window ends.
- **Channel discovery** — finds the current live stream and recent stream replays from an `@handle`, channel URL or `UC…` channel ID.
- **Every visible event type** — normal messages, Super Chats, Super Stickers, memberships, gift purchases, gift redemptions, paid gift messages, engagement notices and slow-mode/system notices.
- **Replay offsets** — each replay event carries `videoOffsetMsec` and a readable `videoOffsetText` such as `0:05:06`.
- **Numeric money** — `amountText` plus `amountValue` and `currency` when the amount is unambiguous.
- **Author detail** — display name, channel ID, avatar, badges, handle, profile URL, and owner / moderator / member / verified flags.
- **Stream context** — title, channel, live status, start/end time, publish date, duration, views, likes and category.
- **Type filters and caps** — include or exclude event families, cap messages per video and total items, and restrict replay output to a time range.
- **Optional summary** — one `stream_summary` record per video with event counts, per-currency totals, top supporters and a completeness note.

## Inputs

| Field | Default | What it does |
|---|---|---|
| `videoRefs` | — | Watch, live, shorts or `youtu.be` links, bare 11-character IDs, or channel live links such as `@handle/live`. |
| `channelRefs` | — | `@handle`, channel URL or `UC…` ID. Each channel is searched for streams. |
| `mode` | `auto` | `auto` detects live versus replay. `active` forces live chat. `replay` forces saved replay chat. |
| `channelDiscoveryMode` | `current_live_then_replays` | `current_live`, `current_live_then_replays` or `latest_replays`. |
| `maxVideosPerChannel` | `5` | How many streams to take from each channel. |
| `lookbackDays` | `30` | How far back to include ended streams from a channel. `0` means no limit. |
| `includeUpcomingLives` | `false` | Emit an `upcoming` status row for scheduled channel streams. |
| `maxMessagesPerVideo` | `5000` | Chat events per video. `0` collects every available event. |
| `maxTotalItems` | `0` | Whole-run dataset cap. `0` means no limit. |
| `maxTimePerVideoSecs` | `600` | Safety budget for replay pagination on one video. |
| `includeNormalMessages` | `true` | Include standard text messages. |
| `includeSuperChats` | `true` | Include Super Chats and Super Stickers. |
| `includeMembershipEvents` | `true` | Include memberships. |
| `includeGiftEvents` | `true` | Include gift purchases, redemptions and paid gift messages. |
| `includeSystemEvents` | `true` | Include engagement, slow-mode and other system notices. |
| `includePlaceholders` | `false` | Include empty placeholder rows. |
| `includeSummary` | `false` | Add a stream summary row per video. |
| `startOffsetSecs` / `endOffsetSecs` | `0` / `0` | Replay-only time window inside the video. |
| `liveCollectSecs` | `60` | How long to poll an active chat. |
| `pollIntervalMs` | `2000` | Minimum live poll delay; YouTube's longer timeout is honoured. |
| `language` / `regionCode` | `en` / `US` | Page and chat request locale. |
| `includeMetadata` | `true` | Repeat stream metadata on each chat row. |

## Output

Every chat record has `recordType: "live_chat_message"` and:

- identity and timing: `videoId`, `videoUrl`, `chatType`, `messageId`, `eventType`, `messageType`, `timestamp`, `timestampUsec`, `publishedAt`, `videoOffsetMsec`, `videoOffsetText`, `scrapedAt`
- author: `authorName`, `authorChannelId`, `authorHandle`, `authorPhotoUrl`, `authorProfileUrl`, `authorBadges`, `memberBadge`, `isOwner`, `isModerator`, `isMember`, `isVerified`
- content: `messageText`, `messageParts[]` (text and emoji runs)
- money and membership: `amountText`, `amountValue`, `currency`, `superChatColor`, `stickerName`, `stickerUrl`, `membershipText`, `membershipLevel`, `memberMonths`
- gifts: `giftCount`, `giftName`, `giftImageUrl`, `giftedBy`
- system: `systemAction`, `systemText`
- stream context when `includeMetadata` is on: `videoTitle`, `channelName`, `channelId`, `channelUrl`, `channelHandle`, `isLiveContent`, `isLiveNow`, `streamStatus`, `streamStartedAt`, `streamEndedAt`, `videoPublishedAt`, `videoDurationSeconds`, `videoViewCount`, `videoLikeCount`, `videoCategory`
- channel discovery context when the target came from a channel: `discoverySource`, `channelRef`, `isDiscoveredLive`, `isDiscoveredUpcoming`, `discoveredAt`

Event values include `message`, `super_chat`, `super_sticker`, `membership`, `membership_gift`, `gift_received`, `gift`, `system` and `placeholder`.

When chat cannot be read, the actor emits a `recordType: "status"` row with a specific status such as `chat_disabled`, `upcoming`, `no_replay`, `stream_ended`, `stream_live`, `channel_not_live`, `rate_limited` or `partial_capture`, plus a plain-language `error`.

## Examples

Read a complete replay:

```json
{
  "videoRefs": ["https://www.youtube.com/watch?v=FQbNtLOVlwU"],
  "mode": "replay",
  "maxMessagesPerVideo": 0
}
```

Capture 60 seconds of a live chat:

```json
{
  "videoRefs": ["https://www.youtube.com/live/74WjpIH4jeE"],
  "mode": "active",
  "liveCollectSecs": 60
}
```

Find a channel's current live stream and its two most recent replays:

```json
{
  "channelRefs": ["@CBSNews"],
  "channelDiscoveryMode": "current_live_then_replays",
  "maxVideosPerChannel": 2,
  "lookbackDays": 30,
  "includeSummary": true
}
```

Read only paid events from a replay between two offsets:

```json
{
  "videoRefs": ["1vvDbx3nYpg"],
  "mode": "replay",
  "includeNormalMessages": false,
  "includeMembershipEvents": false,
  "includeGiftEvents": false,
  "includeSystemEvents": false,
  "startOffsetSecs": 60,
  "endOffsetSecs": 600
}
```

## FAQ

**Do I need cookies, an API key or a proxy?**
No. The actor reads public chat data without cookies or credentials. It connects directly and switches proxy automatically only when a request is blocked.

**Why is a live run shorter than the whole chat?**
A live read only sees what appears while the actor is collecting. The actor emits the recent history already on screen, then polls until `liveCollectSecs` ends. Run replay mode after the stream ends for the full timeline.

**How much replay chat can I get?**
As much as YouTube exposes. Pagination stops only at the end of the replay, `maxMessagesPerVideo`, `maxTotalItems`, `maxTimePerVideoSecs`, or a real error. There is no fixed page cap.

**What is the difference between `active` and `auto`?**
`auto` checks the video and chooses live or replay. `active` insists on live chat and reports `stream_ended` if the stream is already over. `replay` insists on saved chat and reports `stream_live` if the stream is still running.

**Why is `amountValue` sometimes missing?**
Only unambiguous amounts are converted to a number and currency. The original `amountText` is always kept. Unusual formats are counted in `amountsNotRead` on the summary row instead of being guessed.

**Are gift memberships the same as memberships?**
No. A purchase is `membership_gift` (the buyer sent memberships), a redemption is `gift_received` (someone received one), and a paid gift message is `gift`. The summary keeps separate counts.

**Why are some messages missing from a channel result?**
Channel discovery only returns streams that match `channelDiscoveryMode`, `lookbackDays` and `maxVideosPerChannel`. Some streams have chat disabled, no saved replay, or are members-only. Each of those cases produces a status row instead of fabricated data.

**Does the actor convert currencies or provide a single total?**
No. Totals stay per currency in `totalAmountByCurrency`; no exchange-rate service is used. Top supporters are ranked by numeric amount and keep their own currency.

**Can it read members-only chat or deleted-message moderation details?**
No. Those require an authenticated channel session. The actor reads public chat, reports unavailable paths as status rows, and never fakes restricted data.

## Limits

- Live captures are partial by design; replay is the complete path.
- Members-only chat, age/region-gated videos and moderation logs are not available without authentication.
- Reply-thread parent IDs and a numeric concurrent-viewer count are not exposed in the public chat payload, so they are not returned.
- Currency conversion is intentionally not performed.
- Chat availability is controlled by the creator. A stream can be public while its chat or replay is disabled.

## Data source

The actor reads YouTube's public watch pages and its public InnerTube live-chat endpoints. It does not log in, post messages, or perform moderation actions.
