---
name: orbio-gateway
description: Use an Orbio key to call models and tools, understand what a call costs before making it, and read the balance. Use when working with ORBIO_API_KEY, the Orbio gateway, CREDIT balance, or any orbio_* / social.* / web.* / chain.* tool.
---

# Working with Orbio

The Claude Code plugin connects through browser sign-in. Use its MCP tools
without asking for an API key or requiring `ORBIO_API_KEY` in the environment.
If authentication is needed, ask the user to open `/mcp`, select Orbio and
Authenticate. Never create, rotate or revoke a gateway key just to connect the
plugin. API keys are a separate option for direct HTTP and SDK integrations.

Orbio is one key and one balance for both model calls and tools. The same
`ORBIO_API_KEY` that talks to a model also reads X, searches the web and reads
Robinhood Chain, and every call settles against the same CREDIT balance.

One CREDIT is one dollar. Balances are exact integers of micro-dollars behind
the scenes, so never do money arithmetic in floating point when an exact figure
is available.

## Before you spend anything

Check what a call costs. `GET /api/v1/tools` is the catalogue, and it is
machine readable:

```bash
curl https://api.orbio.so/api/v1/tools
```

Each entry carries `input_schema`, `output_schema` and a `price` block:

- `credit_per_unit`: what one result, page or call costs.
- `credit_to_start`: what starting the job costs **before any results**. Null
  for most tools. When it is set, it can be most of the bill on a small call,
  so never budget from `credit_per_unit` alone.
- `bounded_by`: the argument that decides how many units you can consume.
- `note`: anything where the rate alone would mislead. Read it.

## Every call takes `max_cost`

`max_cost` is a ceiling in CREDIT, as a string. A call quoted above it is
refused **before anything runs**, so a bad `limit` costs nothing.

```bash
curl https://api.orbio.so/api/v1/tools/social.x.posts \
  -H "Authorization: Bearer $ORBIO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"handle":"orbiodotso","limit":20,"max_cost":"0.01"}'
```

Always set it. It is the difference between a mistake that costs a fraction of
a cent and one that empties a balance.

## Reading the answer

A successful call returns `{ id, tool, cost, result }`, and two headers:

- `X-Orbio-Cost`: what this call charged.
- `X-Orbio-Balance`: what is left afterwards.

Use the headers rather than calling a balance endpoint after every tool call.

`202` means the job outlived the request. You get an id, the result is not
ready, and the charge settles when it finishes. This does not happen on the
X tools, which are plain requests.

## When a call fails

Match on the status, not on the prose:

| Status | Meaning | What to do |
| --- | --- | --- |
| `400` | Your arguments are wrong | Fix them. Retrying identical arguments will not help. |
| `402` | Balance too low | Tell your owner. Do not retry. |
| `404` | No such tool | Read `GET /api/v1/tools`. |
| `409` | Something must happen in the dashboard first | Read the message and relay it to your owner verbatim. It names the page and the action. |
| `422` | The provider refused: no such handle, protected account | Change the input, do not retry as-is. |
| `429` | Too many calls | Back off, honour `Retry-After`. |
| `502` | The provider failed | Retry once, then stop. |

A `409` is the one worth special handling. Nothing is wrong with your call and
no retry will ever succeed, because a person has to do something first. The
body carries a machine-readable `error.code` and a `connect_url`.

## Costs nothing to get wrong

A refused call is not charged. A call that returns no data is not charged. A
handle that does not exist comes back with an `error` field beside the others
rather than failing the whole batch, and is not charged.

## Over MCP

Everything above is also on the MCP server at `https://www.orbio.so/api/mcp`,
where tool names use underscores (`social_x_posts`). The price is in each
tool's description and the shape of the answer is in its output schema. Prefer
MCP when you have it: you get the catalogue automatically and do not have to
build requests by hand.

`orbio_get_balance` reads the balance. `orbio_get_key_status` reads the key.
Only create or replace a key when the user explicitly requests that separate action.
