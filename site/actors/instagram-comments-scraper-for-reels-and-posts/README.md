# Instagram Comments Scraper for Reels & Posts

Pull the comment section of any Instagram post or Reel into a clean table you can export to Excel, CSV or JSON. For every comment you get the text, who wrote it, how many likes it received, when it was posted, and the full reply thread underneath it, plus the @mentions and #hashtags found in the text. Paste in one link or a whole list of posts, set how many comments you want, and run. It is built for marketers, community managers, giveaway hosts and researchers who need to read, count and analyse what an audience is saying.

## What this actor does

- **No login for public posts:** comments are read the way a visitor who is not signed in sees them, so a public post needs no Instagram account and no cookies.
- **Posts, Reels and long lists:** accepts `/p/`, `/reel/` and `/tv/` links, or just the post shortcode. Submit a list of links and get all comments in one dataset; duplicate links are collected only once.
- **Very large comment sections:** choose how many comments to collect per post, up to 100,000. They are saved page by page while the run goes on, so nothing is lost if you stop it.
- **With your own cookies, more:** the replies under each comment, linked back to the parent comment and its author with an optional cap per thread (Instagram shows replies only to a signed-in visitor), and private or age-restricted posts that your account can open. Without cookies the comments are still read.
- **A status for every post:** a post that gave no comments is not silently missing; it gets one record with its link and a status such as `post_not_found` or `post_is_private`, and the run log says why.

## Output per comment

Each dataset item is one comment or one reply.

**Identity**

- `status` – `success` for a collected comment or reply. For a post that gave no comments see "Status of a post that gave no comments" below.
- `commentId` – unique ID of the comment.
- `commentUrl` – direct link to this comment.
- `postUrl` – link to the post or Reel the comment belongs to.
- `postShortcode` – short code of that post (the part after `/p/` or `/reel/`).

**Content**

- `text` – the comment text. Comments without text show a readable placeholder such as `[Image]`, `[GIF:id]`, `[Reel: url]` or `[Photo unavailable]`.
- `commentType` – `text`, `image`, `video`, `reel`, `photo_share`, `album_share`, `gif`, `sticker` or `media`.
- `isGif` – `true` if the comment is a GIF.
- `mediaUrl` – link to the attached media file when one is available (see "Media comments" below); empty otherwise.
- `mentions` – usernames @mentioned in the text.
- `hashtags` – hashtags used in the text.
- `isEdited` – `true` if the comment was edited after it was posted.

**Author**

- `authorUsername` – the commenter's username.
- `authorId` – the commenter's numeric Instagram user ID.
- `authorIsVerified` – `true` if the commenter has a verified badge.
- `authorProfilePic` – link to the commenter's profile picture.

**Engagement and threading**

- `timestamp` – when the comment was posted (UTC, ISO 8601).
- `likesCount` – likes on the comment.
- `replyCount` – number of replies the comment has.
- `isReply` – `true` if the item is a reply to another comment.
- `parentCommentId` – for replies, the ID of the comment being answered (empty for top-level comments).
- `parentCommentAuthor` – for replies, the username of that comment's author (empty for top-level comments).

**Metadata**

- `scrapedAt` – when the record was collected (UTC, ISO 8601).

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `postUrls` | array | – | **Required.** Instagram post, Reel or IGTV links to collect comments from, up to 10,000. Accepts `/p/`, `/reel/`, `/reels/`, `/tv/` and `username/p/shortcode` formats. Duplicate links are merged into one record. A bare shortcode also works. |
| `maxCommentsPerPost` | integer | `100` | How many top-level comments to collect per post (1–100,000). Replies are collected in addition to this number. Collecting 100,000 comments takes about an hour, so give the run enough time. |
| `includeReplies` | boolean | `true` | Also collect the replies under each comment. Instagram lists replies only to a signed-in visitor, so this needs `cookies`; without them the comments are read and the run log says that no replies are read. |
| `maxRepliesPerComment` | integer | `20` | Maximum replies to collect per comment thread (1–1,000). Only used when `includeReplies` is on. |
| `cookies` | string | – | Optional. Your own Instagram cookies as JSON (see Authentication). Needed to read replies, and for posts a visitor who is not signed in cannot open. |

### Status of a post that gave no comments

A post that gave no comments produces one record that holds only its `postUrl` and one of these `status` values. The run log explains which post it was and why.

| `status` | Meaning |
|---|---|
| `post_not_found` | The post was deleted, never existed, or is visible only to signed-in users |
| `post_is_private` | The post is in a private account; it opens only with `cookies` of an account that follows it |
| `post_age_restricted` | The post sits behind an age check; it opens only with `cookies` of an account that is old enough |
| `comments_disabled` | Comments are turned off for the post |
| `no_comments` | The post has no comments, or Instagram shows none of them to a visitor who is not signed in |
| `blocked` | Instagram kept refusing the requests after several tries; run it again later |
| `error` | Something unexpected went wrong; the log has the details |
| `invalid_url` | The entry is not an Instagram post link |

### Example: all comments and replies on one post

```json
{
  "postUrls": ["https://www.instagram.com/p/ABC123xyz/"],
  "maxCommentsPerPost": 100,
  "includeReplies": true,
  "maxRepliesPerComment": 20
}
```

### Example: pick giveaway entries from a Reel (top-level comments only)

```json
{
  "postUrls": ["https://www.instagram.com/reel/DEF456uvw/"],
  "maxCommentsPerPost": 1000,
  "includeReplies": false
}
```

### Example: compare audience reaction across several campaign posts

```json
{
  "postUrls": [
    "https://www.instagram.com/p/ABC123xyz/",
    "https://www.instagram.com/p/GHI789rst/",
    "https://www.instagram.com/reel/DEF456uvw/"
  ],
  "maxCommentsPerPost": 500,
  "includeReplies": true,
  "maxRepliesPerComment": 5
}
```

### Example: a very large comment section

```json
{
  "postUrls": ["https://www.instagram.com/p/ABC123xyz/"],
  "maxCommentsPerPost": 100000,
  "includeReplies": false
}
```

### Example: a private post you can see, with your own session

```json
{
  "postUrls": ["https://www.instagram.com/p/GHI789rst/"],
  "maxCommentsPerPost": 500,
  "cookies": "<your cookies JSON>"
}
```

## Authentication

Public posts need no cookies for their comments: Instagram shows them to any visitor, and the actor reads them that way. Replies are different: Instagram lists them only to a signed-in visitor, so with `includeReplies` on and no cookies the run log says "Replies are not read: reading them needs your Instagram cookies, and none were given" and the comments are read as usual.

Cookies are for posts a visitor who is not signed in cannot open: a private account you follow, or a post behind an age check. Export your instagram.com cookies as JSON from a browser where you are logged in (any cookie-export extension will do) and paste them into the `cookies` field. The field is encrypted when the input is saved. A post whose replies you want, and a post that needs a sign-in, is read in one signed-in browser with your cookies (comments and replies together). In this signed-in read at most 10,000 comments are read per post. If your cookies cannot be used, the comments of a public post are still read without a sign-in, and only the replies are missing.

The actor has no account of its own. If your cookies are rejected, expired or signed out, the run log says so, and the posts that needed them keep their status.

## Good to know

- Each post is checked first: a deleted, private, age-restricted or comment-less post gets its status at once, and the other posts carry on.
- Several posts are read at the same time, so a long list of links finishes much sooner than one post after the other. Comments are saved to the dataset while the run goes on, and the console status message shows how many have been read.
- Instagram does not show every comment of a large post to a visitor who is not signed in, so the number you get can be lower than the number Instagram reports for the post. The run log shows both numbers when they differ.
- Every record has all its fields. A value that does not exist for a comment, such as `mediaUrl` on a text comment or `parentCommentId` on a top-level comment, is an empty string.

## Example output

```json
{
  "status": "success",
  "commentId": "17843670504683182",
  "text": "Congrats! 🎉 @friendname check this out",
  "commentType": "text",
  "isGif": false,
  "mediaUrl": "",
  "authorUsername": "user123",
  "authorId": "427553890",
  "authorIsVerified": false,
  "authorProfilePic": "https://scontent.cdninstagram.com/...",
  "timestamp": "2025-05-01T14:32:10",
  "likesCount": 12,
  "replyCount": 3,
  "isReply": false,
  "parentCommentId": "",
  "parentCommentAuthor": "",
  "isEdited": false,
  "mentions": ["friendname"],
  "hashtags": [],
  "postUrl": "https://www.instagram.com/p/ABC123xyz/",
  "postShortcode": "ABC123xyz",
  "commentUrl": "https://www.instagram.com/p/ABC123xyz/c/17843670504683182/",
  "scrapedAt": "2025-12-04T13:06:03.499460"
}
```

## Media comments

Instagram comments can carry a GIF, photo, video, sticker or a shared post or Reel instead of (or next to) text. The `commentType` field always tells you which kind of comment you are looking at, and `text` holds a readable placeholder when the comment has no words. A direct file link is available only for GIFs. Instagram's own website shows no file for a photo that someone attached to a comment, so such a comment has `[Image]` in `text` and an empty `mediaUrl`; do not rely on `mediaUrl` being filled for every media comment.

| Comment | `text` | Links |
|---|---|---|
| Regular text comment | Full comment text | none |
| GIF | `[GIF:id]` | `mediaUrl` holds the GIF file |
| Photo or video attachment | `[Image]`, `[Video]`, or Instagram's own wording such as `[Photo unavailable]` | `mediaUrl` only when Instagram provides it |
| Shared Reel or post | `[Reel: url]` or `[Post: url]` | the link of the shared item is in `text` |

A comment that has no text and no recognisable attachment is labelled `image`, because that is the most common reason for an empty comment. It is a best guess: Instagram does not say what the attachment is.

## Use cases

- **Social media managers:** export the comments under a campaign post to measure sentiment and spot recurring questions or complaints.
- **Giveaway hosts:** list everyone who commented, with their @mentions, to check entry rules and draw winners.
- **Community moderators:** scan your own posts for spam, abuse and off-topic replies, including those buried in reply threads.
- **Brand and PR teams:** follow how an audience reacts to an announcement, and which hashtags and @mentions keep appearing.
- **Influencer vetting agencies:** judge whether an influencer's comment section shows real conversation by looking at reply depth, likes per comment and repeat commenters.
- **Academics and journalists:** build a dataset of public reactions to a post or a viral Reel for research or reporting.

## Responsible use

A comment section is made of messages written by individuals, and their usernames, profile pictures and opinions are personal data even though the post is public. Before you export or analyse comments, make sure you have a lawful basis for doing so (for example under the GDPR), keep the dataset only as long as you need it, and respect people's requests to have their comments removed. Do not use the results to profile, track or target minors, or to infer sensitive details about commenters such as health, religion, political views or sexual orientation.

## FAQ

### Do I need an Instagram account to use this actor?

No, not for public posts. Instagram shows their comments to any visitor. You need an Instagram account only for posts a visitor cannot open, such as a private account you follow: paste your own cookies as described in the Authentication section.

### How many comments can I collect per post?

Up to 100,000 top-level comments per post, set with `maxCommentsPerPost`. Replies are collected on top of that number when `includeReplies` is on, so a post can return more records than the limit, but never more than 200,000 comments and replies together per post. Posts with more comments are cut off at the limit. Collecting 100,000 comments takes about an hour.

### How do I get only the top-level comments, or the replies as well?

Set `includeReplies` to `false` for one record per top-level comment and no reply threads. This is also what happens when you leave `cookies` empty: Instagram shows reply threads only to a signed-in visitor, so without your cookies the actor reads the comments and logs that no replies are read. Give your own cookies and set `includeReplies` to `true` to get the replies. Even then a thread can hold fewer replies than the comment reports, because Instagram lists only part of some threads.

### How do I tell replies from regular comments, and what does `maxRepliesPerComment` do?

Replies have `isReply` set to `true`, and their `parentCommentId` and `parentCommentAuthor` point to the comment they answer, so you can rebuild the whole thread in a spreadsheet. `maxRepliesPerComment` limits the replies collected from each thread: with a value of `5`, a comment with 40 replies returns 5 of them. The default is `20`; raise it up to 1,000 to read longer threads.

### Can I collect comments from private accounts?

Only with the cookies of an account that follows the private account. Without cookies the post gets the status `post_is_private`; with cookies of an account that cannot see it, it keeps that status and the log says so.

### Why did I get fewer comments than the post shows?

Instagram does not show every comment of a large post to a visitor who is not signed in. The actor collects everything Instagram offers, and the run log shows how many comments were read next to the number Instagram reports.

### What if a post has no comments in my results?

Look at the `status` of the record for that post. `post_not_found`, `post_is_private` and `post_age_restricted` mean the post could not be opened without a sign-in, `comments_disabled` and `no_comments` mean there was nothing to read, and `invalid_url` means the entry was not a post link. The run log says why for each post.

### Do I need to set up a proxy?

No. The actor chooses a working connection to Instagram automatically. Just fill in the input and run it.

### How fresh is the data?

Comments are read live when the run happens, so likes, reply counts and new comments reflect the post at that moment.

### Is this affiliated with Instagram or Meta?

No. It is an independent tool that works with publicly available comment data, and it is not affiliated with, endorsed by or sponsored by Instagram or Meta Platforms, Inc.
