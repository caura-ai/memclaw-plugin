# Changelog

## 1.1.0 (2026-03-31)

- Fix: Remove unsupported bash-style fallback syntax from `.mcp.json`
- Fix: Remove MEMCLAW_URL userConfig (hardcode default URL)
- Fix: Use correct plugin command namespacing (`/memclaw:memclaw-setup`)
- Fix: Setup command writes API key to credentials file directly
- Fix: Prevent shell/JSON injection via `jq --arg` (no shell interpolation of user input)
- Fix: Require `jq` as hard dependency (no vulnerable python3 fallback)
- Fix: Allowlist-based email validation (letters, digits, `@`, `.`, `+`, `-`, `_` only)
- Fix: Use `curl -sS` with `--connect-timeout`/`--max-time` and HTTP status code parsing
- Fix: Specify `openssl rand -hex 3` for retry suffix generation
- Fix: Remove `display_name` from registration payload (privacy)
- Fix: Error handling on both branches of credential write (existing and new file)
- Fix: Add auth error troubleshooting to SKILL.md
- Fix: Correct install command format in README
- Fix: Honest onboarding flow — no misleading "automatic" claims

## 1.0.0 (2026-03-31)

- Initial release
- MCP server connection via Streamable HTTP transport
- `/memclaw-setup` command for zero-friction onboarding
- Memory skill for proactive context storage and retrieval
