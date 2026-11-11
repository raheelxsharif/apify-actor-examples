# Instagram Profile Scraper - Bio & Posts

Look up one or many public Instagram accounts and get a clean record for each: display name, bio, follower and following counts, verified flag and bio links. Add recent posts with captions, hashtags, mentions, likes, comments, media links, location and music, then export everything to Excel, CSV or JSON. Built for marketers, agencies, analysts and researchers who need to size up creators, competitors and brands quickly.

## What this actor does

- **Many accounts per run** — enter a list of usernames, with or without the `@`, and each one is processed in turn
- **Profile snapshot** — name, bio, follower and following counts, verification and bio links
- **Recent posts on demand** — up to 500 posts per profile, covering photos, videos, reels, carousels and IGTV, with no sign-in for public profiles
- **Date and pin filters** — keep only posts newer than a date, and leave out the posts a profile has pinned
- **Profile-only mode** — set `maxPosts` to `0` to skip posts and collect just the account details

## Output per profile

When `maxPosts` is `0` (or the account has no posts to show), each username produces one record with a single `author_meta` object. The same object is attached to every post record of that account.

`author_meta` fields:

- `username` — Instagram username
- `full_name` — display name
- `biography` — bio text
- `profile_pic_url` — profile picture URL
- `profile_url` — link to the profile on Instagram
- `is_verified` — `true` if the account has a verified badge
- `is_private` — `true` if the account is private
- `followers_count` — number of followers
- `following_count` — number of accounts followed
- `external_urls` — links listed in the bio (empty list if none)

## Output per post

One record per post when `maxPosts` is greater than `0`.

- `post_url` — direct link to the post
- `status` — `success`, `post_not_found`, `post_age_restricted`, `post_is_private`, `error`, `blocked` or `invalid_url`
- `author_meta` — the profile object described above
- `shortcode` — Instagram's short ID for the post
- `post_id` — Instagram's numeric media ID
- `username` — username of the post author
- `caption` — full caption text
- `hashtags` — hashtags found in the caption
- `mentions` — @mentions found in the caption
- `tagged_users` — accounts tagged in the post
- `collaborators` — accounts listed as co-authors
- `caption_is_edited` — `true` if the caption was edited after posting
- `pub_date` — publication date and time (ISO 8601)
- `media_type` — `Photo`, `Video`, `Carousel`, `Reel` or `IGTV`
- `media_count` — number of images / videos in the post
- `thumbnail_url` — cover image URL
- `media_items` — every image or video in the post, in order, each with `index`, `type`, `url`, `width` and `height`; videos also carry `duration` and `has_audio`
- `accessibility_caption` — Instagram's automatic alt-text for the media
- `like_count` — number of likes
- `comment_count` — number of comments
- `likes_hidden` — `true` if the author hid the like count
- `comments_disabled` — `true` if comments are turned off
- `location` — tagged location, when there is one
- `music` — audio attribution, when there is one (mostly reels)
- `scraped_at` — when the record was collected (ISO 8601)

Only `success` posts carry the full set of fields. Posts with any other status contain just `post_url`, `status` and `author_meta`. The `status` values mean:

- `success` — the post was collected in full
- `post_not_found` — the post no longer exists or is not available
- `post_age_restricted` — Instagram restricts the post to older viewers
- `post_is_private` — the post belongs to a private account
- `blocked` — Instagram refused the request for that post
- `invalid_url` — the post link was not a valid Instagram post URL
- `error` — something else went wrong while collecting that post

A profile that could not be scraped gets a record with a `status` that names the cause and only the `username` and `profile_url` (a private profile also shows the public details above). The explanation is in the run log, one line per profile, followed by a closing `Done:` summary, and the run moves on to the next username. The profile statuses are `profile_not_found`, `profile_is_private`, `profile_age_restricted` (an age check, for example alcohol brands), `profile_restricted` ("unavailable for certain audiences"), `cookies_unusable` (a private or restricted profile needed your cookies and Instagram did not accept them), `blocked` and `error`; a scraped profile is `success`. The post statuses are listed under Output per post. While the run is going, the status message shows a live count such as `Scraped 2/5 profiles`.

### Profile field availability

The profile details are read the way a visitor who is not signed in sees them. Instagram shows that visitor the name, bio, picture, verified and private flags, the exact follower and following counts and the bio links. The post count is in the page description: exact up to 9,999 posts, and rounded to whole thousands from 10,000 posts (Instagram writes "32K Posts", so `posts_count` is 32000). It does not show the business flag, the category or a business email and phone, so those are not in the record. A private profile shows the same details.

| `author_meta` field | Available |
|---|---|
| `username`, `full_name`, `biography`, `profile_pic_url`, `profile_url`, `is_verified`, `is_private`, `followers_count`, `following_count`, `external_urls` | Yes |
| `posts_count` | Yes. When the profile has 10,000 posts or more, an approximate post count is given instead of the exact one: Instagram only gives a figure rounded to whole thousands (32K becomes 32000). Below 10,000 it is exact. It comes from a cached copy and can lag a little |
| `is_business`, `category`, `email`, `phone` | No (shown by Instagram only to signed-in visitors) |

## Authentication

Public profiles and public posts need no sign-in, however many posts you ask for: a profile is always read first the way Instagram shows it to a visitor who is not signed in, even when you give cookies. Only a private profile (your account must follow it) and age-restricted posts need a signed-in view, and only your own cookies can give one. Export your instagram.com cookies as JSON from a logged-in browser session using any cookie-export browser extension, then paste the JSON into the optional `cookies` field. The field is a secret input, so the Apify Console hides and encrypts it, and the cookies are used only during your run. Before anything is read with them, the actor asks Instagram whether it recognises the cookies; if they are invalid, expired or signed out, the run log says so. Without cookies those cases are reported with their status. The actor does not sign in with any account of its own.

If your cookies expire mid-run, re-export them from your browser and restart the actor.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `usernames` | array of strings | – (required) | Instagram usernames to scrape, with or without `@`, or profile links such as `https://www.instagram.com/natgeo/`. At least one is needed |
| `maxPosts` | integer | `12` | Posts to collect per profile, from `0` to `500`. Posts left out by the two filters below do not count. `0` returns profile data only |
| `newerThan` | string | – | Optional date (`YYYY-MM-DD`). Posts published before it are skipped. Empty means no date filter. It must be from 2000-01-01 up to the day before the run, otherwise it is ignored with a warning in the log. The date is read from the post's id, so it can be off by up to a day |
| `skipPinnedPosts` | boolean | `false` | Leave out the posts a profile has pinned to the top of its grid. Instagram does not mark pinned posts for a visitor who is not signed in, so they are recognised by their place and date; a pinned post that is also the profile's newest post cannot be told apart and stays in |
| `cookies` | string (secret) | – | Optional. Your own Instagram cookies as JSON, used for private profiles your account follows and age-restricted posts. See Authentication |

### Example: quick look at one brand

```json
{
  "usernames": ["nike"],
  "maxPosts": 12
}
```

### Example: compare several publishers side by side

```json
{
  "usernames": ["natgeo", "@nytimes", "bbcnews"],
  "maxPosts": 30
}
```

### Example: account details only for a vetting list

```json
{
  "usernames": ["nasa", "spacex", "esa"],
  "maxPosts": 0
}
```

### Example: deep pull with your own cookies

```json
{
  "usernames": ["redbull"],
  "maxPosts": 500,
  "cookies": "<your cookies JSON>"
}
```

## Example output

A profile-only record (`maxPosts: 0`); the values are illustrative.

```json
{
  "status": "success",
  "author_meta": {
    "username": "examplebrand",
    "full_name": "Example Brand",
    "biography": "Better ingredients, better you",
    "profile_pic_url": "https://.../profile.jpg",
    "is_verified": true,
    "is_private": false,
    "followers_count": 30213,
    "following_count": 12,
    "profile_url": "https://www.instagram.com/examplebrand/",
    "external_urls": ["https://example.com/shop"]
  }
}
```

## Use cases

- **Influencer marketers** — check follower counts, verification and posting activity before signing a creator
- **Agency account managers** — compare a client's profile and recent posts with competitors in one export
- **Social media analysts** — see which post types, captions and hashtags earn the most likes and comments
- **Partnership teams** — vet creators and brands in a niche by their bio, follower counts and bio links before contacting them individually
- **Brand and PR teams** — audit how tagged collaborators appear on a brand's feed
- **Academic researchers** — gather public captions, hashtags, locations and engagement counts for communication studies

## Responsible use

The data this actor returns is publicly available on Instagram, but bios (which can hold contact details), names, photos and tagged accounts can still identify real people. Before you collect or store them you need a lawful basis (for example under the GDPR), and you should keep only what your purpose requires. Do not use the results to profile, track or target minors, and do not use them to single out people by special categories of personal data such as health, religion, political opinion or sexual orientation. Contact details found on a profile are meant for the business purpose they were published for, not for unsolicited bulk messaging.

## FAQ

**Do I need an Instagram account?**
No, not for public profiles. Only private profiles and age-restricted posts need cookies, and you can paste your own in the optional `cookies` field (see Authentication above).

**Can it scrape private accounts?**
Only with your own cookies, from an account that follows it. Without cookies you get one record with the public profile details and the status `profile_is_private`.

**How many posts can I get per profile?**
Up to 500 per profile in one run. Use `maxPosts` to choose any number from `1` to `500`, or `0` for profile details only. Accounts with fewer posts simply return fewer.

**Which post types are included?**
Photos, videos, reels, carousels and IGTV. For carousels, every slide is listed in `media_items`, in order.

**Why are `email`, `phone`, `category` and `is_business` missing, and is `posts_count` exact?**
The first four are shown by Instagram only to signed-in visitors, and the actor reads profiles without signing in. `posts_count` is exact below 10,000 posts. When a profile has 10,000 posts or more, an approximate post count is given instead: Instagram writes it as an abbreviation such as "32K", and the actor turns it into 32000 (see Profile field availability).

**What happens if a username does not exist or fails?**
That username gets a record with a `status` such as `profile_not_found` or `error` (the log explains it), and the run continues with the rest of your list.

**How current is the data?**
It is collected live when you run the actor, so follower counts, likes and comments reflect that moment.

**Do I need to set up a proxy?**
No. The actor handles its connection to Instagram automatically. Just enter your usernames and run it.

**Is this affiliated with Instagram or Meta?**
No. It is an independent tool that works with publicly available profile data. It is not affiliated with, endorsed by or sponsored by Instagram or Meta Platforms, Inc.
