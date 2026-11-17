---
name: orbio-media
description: Generate and edit images and video with fal through Orbio, paid in CREDIT. Nano Banana images and edits, Seedance 2.5 videos with sound from a prompt, a first frame or image, video and audio references, video edits that keep a clip's motion, and saved characters that appear in all of them. Use when asked to make an image, a video, an ad visual, a character, or to put a product or a person into an existing clip.
---

# Images and video, through Orbio

Every tool here runs on fal on Orbio's account and is paid from your Orbio balance. No
fal account or key is needed. Pick the tool for the job; the model behind it is chosen
for you.

| Job | Tool |
| --- | --- |
| A photograph, an illustration, a product shot, or an edit of your images | `media.image` |
| A video with sound, from a prompt, a first frame, or image, video and audio references | `media.video` |
| Change who or what is in a video, or its look, keeping its motion | `media.motion` |
| A person you can put into any of the above | `media.character` |
| Collect a job, or check where it is | `media.status` |
| See the looks `media.image` and `media.motion` restyle can take | `media.styles` |

## Images

`media.image` waits for its own result, usually under half a minute, and answers with
`images[].url`. `quality` picks the model: `standard` (the default) is Nano Banana 2, good
at photographs, illustrations and legible text, and edits or combines up to 10 `images`
you send; `fast` is Z-Image Turbo, about a second, for drafts and quick visuals, text to
image only; `best` is Nano Banana Pro, for complex design and brand work. `resolution` is
`1K`, `2K` or `4K`.

## Video

`media.video` makes 4 to 30 seconds with sound at 480p, 720p or 1080p, three ways:

- from a `prompt` alone;
- from a `first_frame` image, and optionally a `last_frame`;
- from references: up to 30 `images`, 10 `videos` and 10 `audio` clips, which the prompt
  names as `@Image1`, `@Video1`, `@Audio1` and so on, for a subject, a motion, a look or a
  soundtrack to follow. Reference videos and audio run 1.8 to 30.2 seconds each, and 30.2
  in all.

`quality: "fast"` is Seedance 2.5 Flash: about two thirds of the price, 480p or 720p, and
a third of the reference slots.

## Collecting results

A video or a motion edit takes minutes, so the call answers `202` with `status: "running"`,
an `id` and `collect_with: "media.status"`. A slow image does the same. Call `media.status`
with that `id` every `retry_after_seconds` until `status` is `completed` (the result is in
`images` or `video`) or `failed` (`error` says why). `media.status` is free and works at
any balance, including zero.

A job that fails, or that moderation refuses, is never charged. Outputs are kept at least
seven days (`expires_at`); download anything worth keeping.

## Characters

`media.character` saves a person from 1 to 6 photo URLs: clear photos of the same face from
different angles. Use photos of yourself, or of someone who agreed to it, and never of
anyone else. There is no training, so it is ready at once; its `id` is the character's id.
Pass it as `character` to `media.image`, `media.video` or `media.motion`.
`media.characters` lists yours.

## Video edits

`media.motion` takes a source `video`: an https MP4 of 1.8 to 30.2 seconds that downloads
directly. `transfer` has the person or product in `images` perform the source's motion,
`swap` replaces an object in the shot with the one in `images` (say which in `prompt`), and
`restyle` renders the clip in a `style` from `media.styles`. Orbio measures the source
before anything is held, so the hold is the most it can cost.

## Local files

Any image field takes https URLs or base64 `data:` URLs of a JPEG, PNG or WebP, so a photo
on disk can be sent inline. A whole call is limited to about 4 MB over the API and 1 MB
over MCP, so send small images inline and host larger ones. Videos and audio must be https
URLs; Orbio copies a video it is given, up to 100 MB.

## Posting what you made

A result URL can go straight into `social.post` as `media`, so an agent can make a video and
post it in two calls.

## What it costs

| Call | CREDIT |
| --- | --- |
| `media.image`, per image | 0.088 standard at 1K (0.132 at 2K, 0.176 at 4K), 0.0055 fast, 0.165 best |
| `media.video`, 5 seconds, at most | 2.589142 at 720p, 6.344678 at 1080p, 1.151633 at 480p, 1.693831 fast at 720p |
| `media.motion`, a 10 second source, at most | 10.378771 at 720p, usually about two thirds of that |
| `media.character`, once | 0.022 |
| `media.status`, `media.characters`, `media.styles` | free |

`media.video` costs the same with or without sound, image references or audio. A reference
video adds its own seconds and bills the whole call at about 0.6 of the rate, though it is
held at the full rate. Each call is held at the most it can cost and charged what fal bills
for the work, never above that hold. Pass `max_cost` to cap
any call; a call quoted above it, or above what the balance can hold, is refused before
anything runs.
