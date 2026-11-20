---
title: Quickstart
description: Connect Kviria to your agent in two minutes via MCP or HTTP.
---

## 1. Connect or get a key

- **MCP clients (Claude, ChatGPT, Cursor, Claude Code):** No key needed. Add `https://api.kviria.com/mcp` and approve access in one click via OAuth. See [MCP server](/api/mcp).
- **HTTP API & SDKs:** Open [api.kviria.com/key](https://api.kviria.com/key) and click **Create a key**. Copy the `kvr_...` key (shown once). Every account includes `600` free turns per day and `60` turns per minute. See [Rate limits and errors](/api/limits).

## 2. Make your first call

Send the user's message with `request: "ask"` and `words`. Carry the returned `state` into the next turn.

### Turn 1: Area search (no GPS pin needed)

```bash
curl -X POST "https://api.kviria.com/v1/agent?agent=my-agent" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $KVIRIA_KEY" \
  -d '{
    "request": "ask",
    "words": "a cafe in Williamsburg, Brooklyn",
    "state": ""
  }'
```

Trimmed response (`decision: "many"`):

```json
{
  "decision": "many",
  "distinguishBy": "street",
  "candidates": [
    {
      "gers": "08f2a100d8cb5d1a0390a5e1b5c6d701",
      "name": "Devocion",
      "category": "cafe",
      "confidence": 0.96,
      "location": { "lat": 40.71602, "lng": -73.96134 },
      "address": { "number": "69", "street": "Grand St" }
    },
    {
      "gers": "08f2a100d8cb5d1a0390a5e1b5c6d702",
      "name": "Oslo Coffee Roasters",
      "category": "cafe",
      "confidence": 0.95,
      "location": { "lat": 40.71645, "lng": -73.95891 },
      "address": { "number": "133", "street": "Roebling St" }
    }
  ],
  "area": {
    "id": "area-williamsburg-ny",
    "name": "Williamsburg",
    "level": "neighborhood",
    "parent": ["New York", "New York", "Brooklyn"],
    "region": "US-NY"
  },
  "evidence": [{ "test": "area", "result": "pass", "reasonCode": "area_resolved" }],
  "state": "AXbU2v9QmZ3k8sJt0LrE4wHn7cYq1pKf6gNa5oTd"
}
```

### Turn 2: Pick an option

Pass the user's choice back with the returned `state`:

```bash
curl -X POST "https://api.kviria.com/v1/agent?agent=my-agent" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $KVIRIA_KEY" \
  -d '{
    "request": "ask",
    "words": "the first one",
    "state": "AXbU2v9QmZ3k8sJt0LrE4wHn7cYq1pKf6gNa5oTd"
  }'
```

Trimmed response (`decision: "one"`):

```json
{
  "decision": "one",
  "target": {
    "gers": "08f2a100d8cb5d1a0390a5e1b5c6d701",
    "name": "Devocion",
    "category": "cafe",
    "confidence": 0.96,
    "location": { "lat": 40.71602, "lng": -73.96134 },
    "address": { "number": "69", "street": "Grand St" }
  },
  "anchor": { "lat": 40.71602, "lng": -73.96134 },
  "evidence": [{ "test": "resolve", "result": "pass", "reasonCode": "picked_candidate" }],
  "state": "BYcV3w0RnA4l9tKu1MsF5xIo8dZr2qLg7hOb6pUe"
}
```

### Around a user's position (`anchor`)

When you have the user's GPS coordinate, send `anchor` on the first turn instead of an area:

```bash
curl -X POST "https://api.kviria.com/v1/agent?agent=my-agent" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $KVIRIA_KEY" \
  -d '{
    "request": "ask",
    "words": "is there a coffee shop near here",
    "anchor": { "lat": 37.76326, "lng": -122.4152 },
    "state": ""
  }'
```

### TypeScript / Node.js

```javascript
async function askKviria(words, state = "", anchor) {
  const res = await fetch("https://api.kviria.com/v1/agent?agent=my-agent", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      Authorization: `Bearer ${process.env.KVIRIA_KEY}`,
    },
    body: JSON.stringify({
      request: "ask",
      words,
      state,
      ...(anchor ? { anchor } : {}),
    }),
  });
  return res.json();
}
```

### Python

```python
import os
import requests

def ask_kviria(words: str, state: str = "", anchor: dict | None = None) -> dict:
    body = {"request": "ask", "words": words, "state": state}
    if anchor is not None:
        body["anchor"] = anchor
    res = requests.post(
        "https://api.kviria.com/v1/agent",
        params={"agent": "my-agent"},
        headers={"Authorization": f"Bearer {os.environ['KVIRIA_KEY']}"},
        json=body,
        timeout=20,
    )
    return res.json()
```

## 3. Give your agent the tool and system prompt

1. Load the tool schema from [kviria.com/tool-definition.json](https://kviria.com/tool-definition.json).
2. Give your agent the system prompt from [kviria.com/skill.md](https://kviria.com/skill.md). It instructs the model to call `kviria` on every location turn, pass `state` unchanged, and reply strictly from returned facts.
