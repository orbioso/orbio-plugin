# Orbio for Claude Code

One key for model calls and agent tools, billed against the same CREDIT balance.

```bash
/plugin marketplace add orbioso/orbio-plugin
/plugin install orbio@orbio
export ORBIO_API_KEY=orb_...
```

That is the whole setup. The plugin brings:

- **The MCP server** at `api.orbio.so/api/mcp`, so every tool arrives with its
  price, its arguments and the shape of its answer. No wrapper code.
- **Two skills.** `orbio-gateway` covers the key, the balance, what a call costs
  before you make it, and what each failure means. `orbio-social` covers reading
  X and publishing.
- **`/orbio-costs`**, which prints the live catalogue and your balance.

Get a key at [orbio.so/dashboard](https://orbio.so/dashboard), or ask the MCP
server for one with `orbio_create_key` once you are connected.

## What you can do with it

Call any model, read X (search, timelines, mentions, reply trees, profiles),
search and scrape the web, read Robinhood Chain, scrape Instagram, TikTok and
Reddit, and publish to social accounts.

Every call takes `max_cost` in CREDIT and is refused before it runs if the
quote is above it, so a mistaken `limit` costs nothing.

## Publishing needs a connected account

Reading needs only a key. Publishing acts as you on an account you own, so it
is authorised in the dashboard at [orbio.so/dashboard#tools](https://orbio.so/dashboard#tools),
signed in to the account whose gateway key the agent uses. A key can post once an account
is connected; a key can never connect one. Orbio stores an account id and never
a social credential.

## Writing code instead

The plugin is for a model working in Claude Code. To have an agent pay for
itself from your own code, including buying CREDIT and the launchpad, install
[`@orbiodotso/sdk`](https://www.npmjs.com/package/@orbiodotso/sdk). It holds
the wallet key in your process and signs locally.

## Docs

- Tools, prices and schemas, live: `GET https://api.orbio.so/api/v1/tools`
- API reference: [orbio.so/launchpad/docs](https://orbio.so/launchpad/docs)
- MCP setup for other clients: [orbio.so/mcp](https://orbio.so/mcp)

MIT.
