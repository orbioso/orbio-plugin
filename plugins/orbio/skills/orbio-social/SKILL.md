---
name: orbio-social
description: Read X (search, timelines, mentions, reply trees, profiles) and publish posts to social accounts through Orbio. Use when asked to read tweets, track mentions, measure engagement, or post to X, LinkedIn, Instagram, TikTok or other platforms.
---

# Reading and posting on social, through Orbio

Reading uses your authenticated Orbio connection. Publishing needs an account your **owner**
connected for you, and you cannot connect one yourself.

## Reading X

`social.x.posts` covers four shapes through one tool. Give exactly one of:

| Argument | Gets you |
| --- | --- |
| `handle` | That handle's posts and replies |
| `mentions_of` | Posts mentioning that handle, excluding its own |
| `conversation_id` | The reply tree under a post id |
| `query` | A raw X search, with any X search operator |

Plus `sort` (`Latest` or `Top`), `limit` (up to 100) and `cursor`.

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

Accounts X has shadow-banned return **zero results** from search, including
from `handle`. That is not the same as "this account is inactive". If you get
zero and you expected posts, say the result was empty rather than concluding
the account is dormant.

## Reading profiles

`social.x.profile` takes `handles` and returns bio, follower counts, join date
and verification. A handle that does not exist comes back with an `error` field
instead of the rest, and is not charged, so a batch never fails because one
name was wrong.

## Publishing

`social.post` takes `text` and optionally `platforms`. With no `platforms` it
posts to every connected account.

```json
{ "text": "Shipped v2. Fees now compound hourly.", "platforms": ["twitter"], "max_cost": "0.05" }
```

It returns `post_id`, `status` and a `platformPostUrl` per platform once live.
`social.post.status` follows a post afterwards and is free.

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

### Posting to X costs far more if the text contains a link

X charges to publish, and charges **over thirteen times more** for a post
containing an `http` or `https` link. Orbio passes that through at cost, so:

- a plain post to X costs about **0.0187 CREDIT**
- the same post with a link costs about **0.2222 CREDIT**

This is X's pricing, not a markup. If you are posting many links, that is a
real budget, and `max_cost` is quoted from your text, so a link post is refused
against a ceiling set for a plain one. If a link is not essential, leaving it
out is a large saving. Other platforms carry no per-post charge.

### Publishing is immediate and cannot be undone

`social.post` publishes now. There is no draft and no scheduling through this
tool, and nothing takes a post back once a platform has it. Treat it as you
would any irreversible action: if the text was not given to you explicitly,
show it to your owner before sending it.

Retries are safe. Each call carries an idempotency key, so a call you retry
because you never saw the answer returns the original post rather than posting
twice. Retrying with *different* text is a different post.

## What runs behind these

Reads come from a REST provider that answers in about a second. Instagram,
TikTok and Reddit scraping still runs as a job and can take longer, answering
`202` with an id when it outlives the request. The tool names never change when
a provider does, so build against the names.
