---
title: MCP server
description: Connect Kviria to Claude, ChatGPT, Cursor, or any MCP client over streamable HTTP.
---

```text
https://api.kviria.com/mcp
```

Kviria's Model Context Protocol (MCP) server exposes one tool, `kviria`, over streamable HTTP. It accepts the exact same arguments and returns the exact same JSON decisions as [`POST /v1/agent`](/api/agent).

## Connect with OAuth (one click)

MCP clients that support OAuth connect without an API key:

- **Claude.ai / ChatGPT:** Add `https://api.kviria.com/mcp` as a custom connector and click **Allow** on the Kviria connect page.
- **Claude Code:**
  ```bash
  claude mcp add --transport http kviria https://api.kviria.com/mcp
  ```
- **Cursor / VS Code (`mcp.json`):**
  ```json
  {
    "mcpServers": {
      "kviria": {
        "url": "https://api.kviria.com/mcp"
      }
    }
  }
  ```

## Connect with a Kviria key

For headless agents or MCP clients that use static headers, pass a key from [api.kviria.com/key](https://api.kviria.com/key):

```json
{
  "mcpServers": {
    "kviria": {
      "url": "https://api.kviria.com/mcp",
      "headers": {
        "Authorization": "Bearer kvr_..."
      }
    }
  }
}
```

## Notes for MCP callers

- Omit `anchor` on follow-up turns when the user has not moved (do not send an empty `anchor: {}` object).
- Copy the `state` string from the tool result into the next tool call.
- MCP requests do not need the `x-kviria-agent` header.
