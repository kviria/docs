---
title: Overview
description: Location understanding for voice and chat agents across the United States.
---

Kviria finds the exact place, address, or street corner a user means and returns its verified coordinate.

A user talks to your agent in their own words, around their current position (`"a cafe across from the bank"`) or in an area they name (`"espresso in Williamsburg, Brooklyn"`, `"597 Greene Ave, Brooklyn"`). Kviria narrows the conversation turn by turn until one place is settled. It returns structured JSON facts only; your agent speaks the reply.

Connect over HTTP ([`POST /v1/agent`](/api/agent)) or through the [MCP server](/api/mcp).

```text
User:  "a cafe in Williamsburg, Brooklyn"
Call:  { "request": "ask", "words": "a cafe in Williamsburg, Brooklyn", "state": "" }
Reply: decision "many" (Devocion on Grand St, Oslo Coffee on Roebling St, ...)

User:  "the first one"
Call:  { "request": "ask", "words": "the first one", "state": "<STATE_1>" }
Reply: decision "one" (Devocion, 69 Grand St, location { lat: 40.716, lng: -73.961 })

User:  "what is across the street from it?"
Call:  { "request": "ask", "words": "what is across the street from it?", "state": "<STATE_2>" }
Reply: decision "many" (places across Grand St from Devocion)
```

## How a turn works

Every call sends:

- **`request`**: send `"ask"` with the user's message in `words`, or send structured fields (`search`, `find`, `near`, `across`, `next_to`, `corner`, `street_end`, `resolve`, `pick`, `confirm`, `more`).
- **`state`**: the signed `state` string from the previous response (`""` on the first turn).
- **`anchor` or `area`** (on the first turn): the user's `{ lat, lng }` position, or the name of the area they want to search. With `request: "ask"`, Kviria also reads the area directly from `words`.

Every response returns one `decision` and a new `state`:

| Decision | Meaning | Your agent's move |
|---|---|---|
| `one` | The place is settled | State the name, distance, and facts from `target` |
| `one_unconfirmed` | Partial name match (location held until confirmed) | Ask the user to confirm the place (`yes` or `no`) |
| `many` | Multiple places match | Ask which one using the fact in `distinguishBy` (at most 3 options) |
| `unclear` | Needs clarification, a heading, or an area choice (`areaOptions`) | Ask the question or offer the 3 `areaOptions` |
| `none` | Nothing matched in the map data | Tell the user what could not be found and ask for another detail |

## Why agents use Kviria

- **No invented places or coordinates.** Language models guess addresses and distances. Kviria answers strictly from verified map data across the contiguous United States.
- **Built for multi-turn dialogue.** Real users say `"the second one"`, `"what's across the street from it"`, or `"in Williamsburg"`. The signed `state` carries the context across turns without server-side session storage.
- **Confirmation before navigation.** When a name matches only in part, Kviria holds the coordinate (`one_unconfirmed`) until the user confirms.

## Contracts for agents

- **Skill prompt & rules:** [kviria.com/skill.md](https://kviria.com/skill.md)
- **Tool JSON Schema:** [kviria.com/tool-definition.json](https://kviria.com/tool-definition.json)
- **OpenAPI 3.1 spec:** [api.kviria.com/openapi.json](https://api.kviria.com/openapi.json)
