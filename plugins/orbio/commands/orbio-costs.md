---
description: Show what every Orbio tool costs, read live from the catalogue
---

Read the live Orbio tool catalogue and show what each tool costs.

1. Call `GET https://api.orbio.so/api/v1/tools` with `Authorization: Bearer $ORBIO_API_KEY`.
2. Print one row per tool: name, what it does in a few words, the price per unit,
   and the cost to start the job where there is one.
3. Call `orbio_get_balance` (or `GET /api/v1/key`) and print the balance underneath.

Read `credit_to_start` as well as `credit_per_unit`. On a small call the start
fee is most of the bill, and a table that omits it understates the price.

If `$ORBIO_API_KEY` is not set, say so and stop: the catalogue needs a key.
