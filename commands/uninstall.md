---
description: Completely remove MemClaw — MCP server, env var, credentials, and plugin
---

# MemClaw Uninstall

Completely remove MemClaw from this Claude Code instance.

## Instructions

Run each step in order. If a step fails, continue to the next — partial cleanup is better than none.

1. **Remove the MCP server registration:**

```bash
claude mcp remove memclaw -s user 2>/dev/null && echo "MCP server removed" || echo "No MCP server found (already removed)"
```

2. **Remove the environment variable from the shell profile:**

```bash
SHELL_RC="$HOME/.zshrc"
if [ -n "$BASH_VERSION" ] && [ -f "$HOME/.bashrc" ]; then
  SHELL_RC="$HOME/.bashrc"
fi

if grep -q '^export MEMCLAW_API_KEY=' "$SHELL_RC" 2>/dev/null; then
  grep -v '^export MEMCLAW_API_KEY=' "$SHELL_RC" > "$SHELL_RC.tmp" && mv "$SHELL_RC.tmp" "$SHELL_RC"
  echo "Removed MEMCLAW_API_KEY from $SHELL_RC"
else
  echo "No MEMCLAW_API_KEY found in $SHELL_RC"
fi

unset MEMCLAW_API_KEY
```

3. **Remove credentials file entry (if present from older installs):**

```bash
CREDS_FILE="$HOME/.claude/.credentials.json"
if [ -f "$CREDS_FILE" ]; then
  UPDATED=$(jq 'del(.pluginConfigs.memclaw)' "$CREDS_FILE" 2>/dev/null)
  if [ $? -eq 0 ]; then
    # If pluginConfigs is now empty, remove the whole file
    REMAINING=$(echo "$UPDATED" | jq '.pluginConfigs | length' 2>/dev/null)
    if [ "$REMAINING" = "0" ]; then
      rm -f "$CREDS_FILE"
      echo "Removed $CREDS_FILE (no other plugins)"
    else
      echo "$UPDATED" > "$CREDS_FILE"
      echo "Removed memclaw entry from $CREDS_FILE"
    fi
  fi
else
  echo "No credentials file found"
fi
```

4. **Disable the plugin in settings:**

```bash
SETTINGS_FILE="$HOME/.claude/settings.json"
if [ -f "$SETTINGS_FILE" ] && jq -e '.enabledPlugins["memclaw@memclaw"]' "$SETTINGS_FILE" >/dev/null 2>&1; then
  jq 'del(.enabledPlugins["memclaw@memclaw"])' "$SETTINGS_FILE" > "$SETTINGS_FILE.tmp" && mv "$SETTINGS_FILE.tmp" "$SETTINGS_FILE"
  echo "Disabled memclaw plugin in settings"
fi

if [ -f "$SETTINGS_FILE" ] && jq -e '.extraKnownMarketplaces.memclaw' "$SETTINGS_FILE" >/dev/null 2>&1; then
  jq 'del(.extraKnownMarketplaces.memclaw)' "$SETTINGS_FILE" > "$SETTINGS_FILE.tmp" && mv "$SETTINGS_FILE.tmp" "$SETTINGS_FILE"
  echo "Removed memclaw marketplace entry"
fi
```

5. **Remove cached plugin files:**

```bash
CACHE_DIR="$HOME/.claude/plugins/cache/memclaw"
if [ -d "$CACHE_DIR" ]; then
  rm -rf "$CACHE_DIR"
  echo "Removed plugin cache at $CACHE_DIR"
else
  echo "No plugin cache found"
fi
```

6. **Print a summary.** Tell the user:
   - MemClaw has been fully removed
   - They should restart Claude Code for changes to take effect
   - To reinstall later: add the marketplace and run `/memclaw:setup <email>`
