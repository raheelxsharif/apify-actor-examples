# YouTube Channel About, Playlists & Posts Finder

A channel's About page carries more than most people realize — join date, country, external links, total views — and this actor captures all of it in one row per channel, with optional playlist and Community Post snapshots folded in when you want them.

## What this actor does

- Resolves a channel from a handle, a full channel URL, or a raw channel ID, and reads its header plus About panel.
- Returns the channel's external links from its public About page, when enabled.
- Optionally nests a snapshot of the channel's public playlists into the same row.
- Optionally nests a snapshot of the channel's public Community Posts, including images, polls, video attachments, and (on request) comments, into the same row.
- Records which optional surface failed to load rather than guessing or leaving you uncertain about why a field is missing.

## Output per channel

| Field | Description |
|---|---|
| `recordType` | `"channel_profile"` for a normal row, or `"status"` if the channel couldn't be read at all |
| `channelId` | Channel's `UC…` ID |
| `channelName` | Channel display name |
| `handle` | Channel's `@handle` |
| `channelUrl` | Canonical channel URL |
| `description` | Channel description |
| `subscriberCountText`, `subscriberCount` | Subscriber count as displayed, and parsed to a number |
| `videoCountText`, `videoCount` | Video count as displayed, and parsed to a number |
| `totalViewsText`, `totalViews` | Total channel views as displayed, and parsed to a number |
| `country` | Channel's listed country |
| `joinedDateText` | When the channel joined YouTube, as displayed |
| `avatarUrl` | Channel avatar image |
| `isVerified` | Whether the channel has a verification badge |
| `links` | External links from the About page, each with `title` and `url` (when `includeLinks` is on) |
| `playlists` | Snapshot of public playlists (see below), when `includePlaylists` is on |
| `communityPosts` | Snapshot of public Community Posts (see below), when `includePosts` is on |
| `surfaceStatus` | Notes which optional surface (`playlists`, `posts`) returned nothing public or was unavailable |
| `scrapedAt` | When the actor collected the record |

### Nested playlist fields (inside `playlists`)

`playlistId`, `title`, `playlistUrl`, `contentType`, `watchUrl`, `firstVideoId`, `watchEndpointPlaylistId`, `videoCountText`/`videoCount`, `thumbnailSources`, `metadataRowsRaw`, `scrapedAt`.

### Status row (when a channel can't be read)

If a channel's public data can't be loaded at all, the row has `recordType: "status"`, `status: "failed"`, `channelUrl` (as you provided it), and `error` describing what happened, instead of the fields above.

### Nested Community Post fields (inside `communityPosts`)

`postId`, `postUrl`, `authorName`, `contentText`, `publishedTimeText`, `voteCountText`, `commentCountText`, `images`, `videoAttachments` (each with `videoId`, `title`, `url`, `viewCountText`, `publishedTimeText`, `lengthText`), `poll` (`pollChoices`, `pollTotalVotesText`, `pollType`), and `comments` (each with `commentId`, `authorName`, `authorChannelId`, `text`, `publishedTimeText`) when `includePostComments` is on.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `channelUrls` | Array of strings | – (required) | Channel handles, full URLs, or raw channel IDs to look up. |
| `includeLinks` | Boolean | `true` | Include the channel's public external links. |
| `includePlaylists` | Boolean | `false` | Nest a public playlist snapshot into each profile row. |
| `maxPlaylists` | Integer | `20` | Maximum playlists to nest per channel (1–10000) when `includePlaylists` is on. |
| `includePosts` | Boolean | `false` | Nest a public Community Posts snapshot into each profile row. |
| `maxPosts` | Integer | `20` | Maximum posts to nest per channel (1–10000) when `includePosts` is on. |
| `includePostComments` | Boolean | `false` | Nest comments and replies inside each Community Post, when public. |
| `maxPostComments` | Integer | `20` | Maximum comments/replies to keep per post (1–1000) when `includePostComments` is on. |
| `market` | Select | `US` | Country code used for the localized channel page. |

### Example: profile only

```json
{
  "channelUrls": ["@MrBeast"],
  "includeLinks": true
}
```

### Example: profile with playlists and posts

```json
{
  "channelUrls": ["@MrBeast"],
  "includePlaylists": true,
  "maxPlaylists": 10,
  "includePosts": true,
  "maxPosts": 5
}
```

### Example: multiple channels in another market, with post comments

```json
{
  "channelUrls": ["@MrBeast", "@Fireship"],
  "market": "GB",
  "includePosts": true,
  "includePostComments": true,
  "maxPostComments": 20
}
```

## Use cases

- **Creator discovery and partnerships** build a shortlist of channels with audience size, country, and contact links in one row.
- **Market research** compare subscriber, video, and view counts across a set of competing channels.
- **Content teams** check which channels are actively posting Community updates and polls.
- **Outreach and sponsorship scouting** export channel descriptions and external links for a campaign list.
- **Longitudinal trackers** re-run the same `channelUrls` periodically to watch subscriber or view growth over time.

## FAQ

**Which channel identifiers can I use?**
A handle like `@name`, a full channel URL, or a raw `UC…` channel ID.

**Why are some fields missing for a channel?**
A field is only included when YouTube actually published that value — a channel with a hidden subscriber count or no listed country simply won't have those fields, rather than showing a blank or guessed value.

**What does `surfaceStatus` tell me?**
It reports, per optional surface you requested (`playlists`, `posts`), whether that surface had nothing public to show or was unavailable for that channel — so a missing `playlists` array is explained rather than silent.

**Do I need cookies or a login?**
No. Every surface this actor reads — the channel header, About panel, playlists, and Community Posts — is public.

**Why would `includePostComments` return no comments even when enabled?**
Comments on a Community Post are only included when YouTube exposes them publicly on that post; some posts have comments disabled or hidden.

**Can I fetch a channel's entire video catalog with this actor?**
No — this actor focuses on the channel's profile plus optional playlist and post snapshots. Use a dedicated channel-videos actor for a full video catalog.

**What does `market` change?**
It sets the country code used when requesting the channel's localized page, which can affect which language text fields like `joinedDateText` or `videoCountText` are rendered in.

**Is this affiliated with YouTube?**
No. This actor independently reads YouTube's public pages and is not affiliated with or endorsed by YouTube or Google.
