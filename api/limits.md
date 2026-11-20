---
title: Rate limits and errors
description: Account quotas, GET /v1/account, response headers, and error codes.
---

## Quotas and rate limits

Kviria is free in beta. Limits apply per Kviria account:

| Limit | Allowance | Window | Error code on `429` |
|---|---|---|---|
| Daily turns (`POST /v1/agent` & `/mcp`) | `600` turns | Per UTC day (resets at `00:00 UTC`) | `daily_limit_reached` |
| Minute turns (`POST /v1/agent` & `/mcp`) | `60` turns | Per minute | `turn_minute_limit` |
| Account status (`GET /v1/account`) | `60` requests | Per minute | `account_minute_limit` |
| New accounts (`/key` & `/authorize`) | `20` accounts | Per network per UTC day | Shown on the page |

### What counts toward the daily quota

- Every answered turn (`200 OK`) counts as `1` turn, including follow-up picks, confirmations, `more`, and queries outside coverage (`out_of_coverage`).
- Invalid requests refused before execution (`400`), authentication failures (`401`), and oversized payloads (`413`) never count toward your daily quota.
- Calling `GET /v1/account` is read-only and never counts as a turn.

## Check remaining turns (`GET /v1/account`)

Query your account's daily allowance at any time with your Bearer key or OAuth token:

```bash
curl "https://api.kviria.com/v1/account" \
  -H "Authorization: Bearer $KVIRIA_KEY"
```

```json
{
  "freeTurns": {
    "perDay": 600,
    "usedToday": 42,
    "leftToday": 558,
    "resetsAt": "2026-11-21T00:00:00Z"
  }
}
```

## Rate-limit headers and backoff

When a request is rate-limited (`429`), Kviria returns:

| Header | Present on | Meaning |
|---|---|---|
| `Retry-After` | All `429` responses | Seconds to wait before retrying |
| `X-RateLimit-Limit-Day` | `429 daily_limit_reached` | Your account's daily turn allowance (`600`) |
| `X-RateLimit-Remaining-Day` | `429 daily_limit_reached` | Remaining turns for the current UTC day (`0`) |

Never retry in a tight loop. Wait for the duration in `Retry-After` before sending the next request.

## Error codes

| Status | `error` | What your agent should do |
|---|---|---|
| `400` | `invalid_request` | Read `message` (and optional `suggestion`), fix the fields, and call again |
| `400` | `anchor_required` | Ask the user where they are or which area they want to search |
| `400` | `invalid_state` | State was altered; start over with `state: ""` |
| `400` | `unsupported_field`, `invalid_body`, `transcript_removed` | Fix the request body shape; send raw user messages with `request: "ask"` and `words` |
| `400` | `invalid JSON body`, or agent name error | Send valid JSON and an agent name matching `[a-z0-9-]{3,64}` |
| `401` | `unauthorized` | Send a valid `kvr_...` key from [api.kviria.com/key](https://api.kviria.com/key) or connect via OAuth |
| `413` | `payload_too_large` | Keep request body under `16 KB` (`64 KB` on `/mcp`) |
| `429` | `turn_minute_limit`, `account_minute_limit` | Wait for `Retry-After` seconds (`60` requests/min limit) |
| `429` | `daily_limit_reached` | Daily `600`-turn limit reached; wait until `00:00 UTC` (`Retry-After`) |
| `429` | `service_busy` | Wait for `Retry-After` seconds and retry |
| `500` | `Server Error` | Retry after a short pause |

### Error bodies that preserve `state`

The `400` responses for `invalid_request`, `invalid_body`, `unsupported_field`, `transcript_removed`, `anchor_required`, and `invalid_state` return `{ error, message, state }`. Pass that `state` into your next call so the conversation continues without losing context (on `invalid_state`, `state` is `""`).
