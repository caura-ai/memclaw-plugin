# memclaw-claude-code-plugin

Persistent AI memory for Claude Code. Remember decisions, context, and knowledge across sessions.

## Quick Start

1. Install the plugin (once it's in the marketplace):

```
/plugin install memclaw
```

Or install directly from this repo:

```
/plugin marketplace add memclaw-repo https://github.com/caura-ai/memclaw-claude-code-plugin
/plugin install memclaw@memclaw-repo
```

2. When prompted for the API key during install, you can leave it blank — the setup command will configure it for you.

3. Register and get your API key:

```
/memclaw:memclaw-setup your@email.com
```

The setup command registers your account, retrieves an API key, and stores it in your local credentials automatically.

4. Verify: run `/mcp` to confirm the memclaw server is connected.

## What This Plugin Provides

- **MCP server connection** — Connects to MemClaw's Streamable HTTP MCP server for persistent memory storage and retrieval
- **`/memclaw:memclaw-setup` command** — Register with just an email to get an API key
- **Memory skill** — Teaches Claude Code when to proactively store and retrieve memories (architectural decisions, debugging gotchas, project conventions, etc.)

## Manual Install

If you prefer to configure the MCP server directly:

```bash
claude mcp add --transport http memclaw https://memclaw.net/mcp/ -H "Authorization: Bearer YOUR_API_KEY"
```

Get an API key by registering (requires `jq`):

```bash
jq -n --arg tid 'your-at-email-com' --arg em 'your@email.com' '{"tenant_id": $tid, "email": $em}' | \
  curl -sS --connect-timeout 10 --max-time 30 \
    -X POST https://memclaw.net/api/register \
    -H 'Content-Type: application/json' \
    -d @-
```

## Available MCP Tools

Connect to the MCP server and run `/mcp` to see available tools. Tools are discoverable once the API key is configured and the server is connected.

## Documentation

[memclaw.net](https://memclaw.net)

## License

MIT
