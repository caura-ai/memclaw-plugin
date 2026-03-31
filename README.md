# memclaw-plugin

Persistent AI memory for Claude Code. Remember decisions, context, and knowledge across sessions.

## Install

### 1. Install the plugin

```bash
# Add source (one-time)
/plugin marketplace add memclaw https://github.com/caura-ai/memclaw-plugin

# Install
/plugin install memclaw@memclaw
```

When prompted for the API key, leave it blank — the setup command configures it for you.

### 2. Setup

```bash
# Register and configure API key
/memclaw:setup your@email.com
```

### 3. Verify

Run `/mcp` to confirm the memclaw server is connected.

## What This Plugin Provides

- **MCP server connection** — Connects to MemClaw's Streamable HTTP MCP server for persistent memory storage and retrieval
- **`/memclaw:setup` command** — Register with just an email to get an API key
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

## Documentation

[memclaw.net](https://memclaw.net)

## License

MIT
