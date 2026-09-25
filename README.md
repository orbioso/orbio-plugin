# Orbio for Claude Code

Connect your Orbio account for model calls and agent tools, billed against the same $CREDIT balance.

```bash
claude plugin marketplace add orbioso/orbio-plugin
claude plugin install orbio@orbio
claude
```

Then open `/mcp`, select the Orbio plugin server, and choose **Authenticate**.
Sign in with the same Orbio account you use on the launchpad. Installation does
not need an API key; browser sign-in authorizes the connection separately and
leaves existing gateway keys active.

Already installed version 0.1.0? Run `claude plugin marketplace update orbio`,
then `claude plugin update orbio@orbio`, and restart Claude Code before signing in.

The plugin brings:

- **The MCP server** at `www.orbio.so/api/mcp`, so every tool arrives with its
  price, its arguments and the shape of its answer. No wrapper code.
- **Two skills.** `orbio-gateway` covers the key, the balance, what a call costs
  before you make it, and what each failure means. `orbio-social` covers reading
  X and publishing.
- **`/orbio-costs`**, which prints the live catalogue and your balance.

## What you can do with it

Call any model, read X (search, timelines, mentions, reply trees, profiles),
search and scrape the web, read Robinhood Chain, scrape Instagram, TikTok and
Reddit, and publish to social accounts.

Every call takes `max_cost` in CREDIT and is refused before it runs if the
quote is above it, so a mistaken `limit` costs nothing.

## Publishing needs a connected account

Reading uses your connected account. Publishing acts as you on an account you own, so it
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

MIT.
