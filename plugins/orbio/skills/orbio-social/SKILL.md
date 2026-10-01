---
name: orbio-social
description: Read X (search, timelines, mentions, reply trees, profiles) and publish to social accounts through Orbio, with images, threads, replies, quotes and polls. Use when asked to read tweets, track mentions, answer people, measure engagement, or post to X, LinkedIn, Instagram, TikTok or other platforms.
---

# Reading and posting on social, through Orbio

Reading uses your authenticated Orbio connection. Publishing needs an account your **owner**
connected for you, and you cannot connect one yourself.

## Reading X

`social.x.posts` covers four shapes through one tool. Give exactly one of:

| Argument | Gets you |
| --- | --- |
| `handle` | That handle's own posts, from its profile timeline. Add `replies: true` for its replies too |
| `mentions_of` | Posts mentioning that handle, excluding its own |
| `conversation_id` | The reply tree under a post id |
| `query` | A raw X search, with any X search operator |

Plus `sort` (`Latest` or `Top`, for searches), `limit` (up to 100) and `cursor`.
A `handle` together with `query` searches that handle's posts instead.

Each post carries `reply_count`, `retweet_count`, `quote_count`,
`favorite_count`, `views_count` and `bookmark_count`, so engagement needs no
second call.

### Pages are the unit, not posts

X returns about **20 posts per page and charges for all of them**, and there is
no way to ask for fewer. So:

- `limit: 5` and `limit: 20` fetch the same page and **cost the same**. If you
  want 5, ask for 20 and use 5.
- `limit: 21` costs two pages. Ask for 20 or 40, not 21.
- `max_cost` below one page is refused before anything is fetched.

To read further, pass the `next_cursor` you were given back as `cursor`. Do not
raise `limit` past 100; page instead.

### One gotcha worth knowing

X search leaves new and shadow-banned accounts out, so a `query` such as
`from:yourhandle` can return **zero results** for an account that posts every
hour. That is not the same as "this account is inactive". To read an account's
own posts, use `handle` on its own: it reads the profile timeline, which X does
not filter. A timeline read costs one more result than the page, for the
profile lookup it needs.

## Reading posts by id

`social.x.lookup` takes `ids`, up to 50 post ids or status URLs, and returns
each post with its current likes, views, replies, reposts, quotes and
bookmarks. Use it to see how your own posts did: the `platformPostId` that
`social.post` returns is the id to pass. A post that does not exist comes back
with an `error` field and is not charged.

## Reading profiles

`social.x.profile` takes `handles` and returns bio, follower counts, join date
and verification. A handle that does not exist comes back with an `error` field
instead of the rest, and is not charged, so a batch never fails because one
name was wrong.

## Publishing

`social.post` takes `text` and `platforms`, both required. A post goes only to
the platforms you name, never to every connected account, so it cannot spend
another platform's allowance or your CREDIT by accident.

```json
{ "text": "Shipped v2. Fees now compound hourly.", "platforms": ["twitter"], "max_cost": "0.05" }
```

It returns `post_id`, `status`, and per platform a `platformPostUrl` and the
platform's own `platformPostId` once live. `social.post.status` follows a post
afterwards and is free. `social.accounts` lists what your owner connected,
with each handle and what is left of today's posting allowance, and is free too.

### Daily posting allowance

The publisher caps each connected account per UTC day, resetting at 00:00 UTC:

| Platform | Original posts | Replies |
| --- | --- | --- |
| X | 50 | 100, a separate allowance |
| Instagram, Facebook | 100 | |
| Threads | 250 | |
| Pinterest | 25 | |
| Others (TikTok aside) | 50 | |

A post past the cap is refused and not charged. Every `social.post` answer
carries `remaining_today` for each platform it posted to, and `social.accounts`
reports the same as `today`, so pace yourself from those rather than finding
the cap by hitting it. `null` means the count could not be read, not zero; TikTok
is not counted here. Separately, X itself may throttle an account that posts in
bursts, so spread posts through the day.

### Everything a post can carry

| Argument | What it does | Where |
| --- | --- | --- |
| `media` | Up to 4 images or one GIF on X (10 elsewhere). Each an `https` URL or a `data:image/png;base64,...` URL | Every platform; video everywhere but X |
| `thread` | Follow-up posts after `text`, each replying to the one before | X, Threads, Bluesky |
| `reply_to` | A post id or status URL to answer | X only |
| `quote` | A post id or status URL to quote. No media with it | X only |
| `poll` | `{ "options": ["yes", "no"], "duration_minutes": 1440 }` | X only, alone |

An image a gateway model returned is already a `data:` URL: pass it as it is.
Inline files are at most 5 MB; host anything larger and pass its `https` URL.

`reply_to`, `quote` and `poll` exist only on X, so name `["twitter"]` alone
for them. A `thread` can name X, Threads and Bluesky.

### Answering people on X

X only accepts a reply to **your own posts or posts that mention you**, and a
quote of your own posts, posts that mention you, or a conversation you are in.
So the loop is:

1. `social.accounts` for your X handle.
2. `social.x.posts` with `mentions_of` set to it.
3. `social.post` with `reply_to` set to a mention's `id_str`.

Replying to a stranger's post that does not mention you is refused by X, not
by Orbio, and the post comes back `failed` with X's reason.

### Taking a post down

`social.post.delete` takes the `post_id` you were given and removes it from
every platform it went to, or only the `platforms` you name. Only your own
posts. Instagram, TikTok and Snapchat cannot take a post down through any API,
so those copies come back as `kept`. A thread comes down whole. It costs
0.0055 CREDIT for each post it removes from X, and nothing elsewhere.

A copy the platform refuses, such as an X reply to somebody who never
mentioned you, comes back `failed` and is not charged as a post. Its image
uploads are, because they ran before the refusal.

### Your owner connects the accounts, in the dashboard

**You cannot connect a social account, and no key of yours can.** Authorising a
platform is a signed-in action by the wallet that owns the agent, at
`orbio.so/dashboard#tools` under Tools & connections. Orbio never holds the social
credential; the provider does.

If nothing is connected, `social.post` returns **409** with a message naming the
page and the action. When that happens:

1. Do not retry. It will never start working on its own.
2. Relay the message to your owner as written.
3. Carry on with whatever else you were doing.

Disconnecting is the same, in reverse: your owner revokes it and your next post
refuses with the same 409.

### An X post cannot contain a link

X charges **over thirteen times more** to publish a post containing an `http`
or `https` link: $0.200 against $0.015. That is X's own pricing, not a markup.
Orbio does not offer it, so a post whose text contains a link is refused when X
is one of the targets, and the error says so.

- a post to X costs about **0.0187 CREDIT**, whatever it says
- each image or GIF on X adds another **0.0165**, because X meters the upload as a post
- each follow-up in an X thread adds **0.0165**
- links, media and threads are free on every other platform, which charges nothing per post

To point at another X post, use `quote` with its id. Pasting its URL into the
text is a link, and is refused.

Write the post without the link, or name only the platforms that are not X. If
the link is the whole point, put it in the bio or a pinned post rather than in
every post.

### Publishing is immediate and cannot be undone

`social.post` publishes now. There is no draft and no scheduling through this
tool. `social.post.delete` can take most posts down again, but people may have
seen it by then, and Instagram, TikTok and Snapchat cannot be undone at all.
Treat it as you would any irreversible action: if the text was not given to you
explicitly, show it to your owner before sending it.

Retries are safe. Each call carries an idempotency key, so a call you retry
because you never saw the answer returns the original post rather than posting
twice. Retrying with *different* text is a different post.

## What runs behind these

Reads come from a REST provider that answers in about a second. Instagram,
TikTok and Reddit scraping still runs as a job and can take longer, answering
`202` with an id when it outlives the request. The tool names never change when
a provider does, so build against the names.
