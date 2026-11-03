# Paddle MCP server

Give any AI assistant direct access to the [Paddle API](https://developer.paddle.com/api-reference/overview) over the
[Model Context Protocol](https://modelcontextprotocol.io).

This server gives AI assistants like Claude, Cursor, and Windsurf direct access to Paddle API operations: products,
prices, subscriptions, customers, transactions, and more. Paddle hosts it, so there's nothing to install or run. This
repository holds the setup instructions only, with no source code.

Three tools are exposed to your assistant: `search` the API reference, `execute` chained API calls in a sandbox, and
`report_missing_tool` for telemetry. A fourth tool, `show_metrics_chart`, draws a metric as an interactive chart in
clients that support [MCP Apps](https://developer.paddle.com/sdks/ai/paddle-mcp/#mcp-apps).

## Endpoint

A single MCP endpoint. Each host serves one environment:

| Environment | Endpoint                                  | Authentication   |
| ----------- | ----------------------------------------- | ---------------- |
| Live        | `POST https://mcp.paddle.com/mcp`         | OAuth or API key |
| Sandbox     | `POST https://sandbox-mcp.paddle.com/mcp` | API key          |

The examples below add both, as `paddle-live` and `paddle-sandbox`, so you can tell your assistant which one to use.

## Authentication

### Live

There are two ways to authenticate: sign in with OAuth, or pass a Paddle API key as a bearer token.

**OAuth (recommended).** Point your client at the endpoint with no credentials. The first time it connects, a browser
window opens and you sign in to Paddle, so there's no API key to create, copy, or keep in a config file.

- A new connection gets read access by default, limited to what your Paddle user's role already permits.
- Change what a connection can do, or revoke it, from **Paddle > Connectors > MCP** in your
  [dashboard](https://vendors.paddle.com/mcps).
- You may be asked to reauthorize from time to time.

**API key.** Prefer a long-lived credential, or running somewhere a browser can't open? Pass a
[Paddle API key](https://developer.paddle.com/api-reference/about/authentication#create-an-api-key) in the
`Authorization` header on every request:

```
Authorization: Bearer <your-paddle-api-key>
```

Scope the key to the permissions you actually need. They are the only limit on what the assistant can do.

### Sandbox

Sandbox connects with a
[sandbox API key](https://developer.paddle.com/api-reference/about/authentication#create-an-api-key) (prefixed
`pdl_sdbx_`), passed in the `Authorization` header on every request. The sandbox server doesn't support OAuth.

```
Authorization: Bearer <your-sandbox-api-key>
```

Scope the key to the permissions you actually need. They are the only limit on what the assistant can do.

## Connect your client

Drop one of these configurations into your AI client. `paddle-live` carries no credentials, so your client runs the
OAuth flow on first connect. Replace `SANDBOX_API_KEY` with your sandbox key.

To connect to live with an API key instead, give `paddle-live` the same `Authorization` header as `paddle-sandbox`,
with your live key.

### Claude Code

```sh
claude mcp add --transport http paddle-live https://mcp.paddle.com/mcp
claude mcp add --transport http paddle-sandbox https://sandbox-mcp.paddle.com/mcp --header "Authorization: Bearer SANDBOX_API_KEY"
```

Then run `/mcp` inside Claude Code to start the browser sign-in.

### Claude Desktop

In `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "paddle-live": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://mcp.paddle.com/mcp"]
    },
    "paddle-sandbox": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-remote",
        "https://sandbox-mcp.paddle.com/mcp",
        "--header",
        "Authorization: Bearer SANDBOX_API_KEY"
      ]
    }
  }
}
```

### Codex

```sh
codex mcp add paddle-live --url https://mcp.paddle.com/mcp
codex mcp login paddle-live

export PADDLE_SANDBOX_API_KEY="SANDBOX_API_KEY"
codex mcp add paddle-sandbox --url https://sandbox-mcp.paddle.com/mcp --bearer-token-env-var PADDLE_SANDBOX_API_KEY
```

### Cursor

In `~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "paddle-live": {
      "type": "http",
      "url": "https://mcp.paddle.com/mcp"
    },
    "paddle-sandbox": {
      "type": "http",
      "url": "https://sandbox-mcp.paddle.com/mcp",
      "headers": {
        "Authorization": "Bearer SANDBOX_API_KEY"
      }
    }
  }
}
```

### VS Code

Press <kbd>Cmd/Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>P</kbd>, choose **MCP: Open workspace folder MCP configuration**,
then edit `mcp.json`:

```json
{
  "servers": {
    "paddle-live": {
      "type": "http",
      "url": "https://mcp.paddle.com/mcp"
    },
    "paddle-sandbox": {
      "type": "http",
      "url": "https://sandbox-mcp.paddle.com/mcp",
      "headers": {
        "Authorization": "Bearer SANDBOX_API_KEY"
      }
    }
  },
  "inputs": []
}
```

### ChatGPT

ChatGPT connects via Plugins rather than a config file. Custom MCP apps need a paid plan (Plus, Pro, Business,
Enterprise, or Edu) with Developer mode enabled. They aren't available on the free tier.

1. Turn on **Developer mode** under **Settings > Security and login**.
2. Open **Plugins**, then create one and set the **MCP server URL** to `https://mcp.paddle.com/mcp` or
   `https://sandbox-mcp.paddle.com/mcp`.
3. For live, select **OAuth** and sign in with Paddle when prompted. For sandbox, or to use a live API key, set
   **Authentication** to **Custom header** and paste `Authorization: Bearer API_KEY`.
4. In a new chat, choose **Developer mode** from the **Plus** menu and select the app.

For more detailed setup instructions, see the
[full installation guide](https://developer.paddle.com/sdks/ai/paddle-mcp/#install-the-mcp-server).

## Use it

Ask your assistant in plain language, and name the server so it picks the right environment. For example:

```
Using paddle-sandbox, set up a Pro plan at $79/month with a 14-day trial, then show me the products and prices.
```

```
Using paddle-live, investigate the failed payment for sarah@example.com and tell me whether it's recurring.
```

```
Using paddle-live, chart my revenue for the last 3 months by week.
```

For more example prompts and best practices, see the
[Paddle MCP server docs](https://developer.paddle.com/sdks/ai/paddle-mcp/#example-prompts).
