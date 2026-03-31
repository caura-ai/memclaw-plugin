---
description: Set up MemClaw — register with just your email and configure the API key
argument-hint: <email>
---

# MemClaw Setup

Set up MemClaw persistent memory for this Claude Code instance.

## Prerequisites

This command requires `jq` for safe JSON construction and credential storage. Run this check first — if it fails, stop and tell the user to install `jq`:

```bash
command -v jq >/dev/null 2>&1 || { echo "ERROR: jq is required. Install: brew install jq (macOS) or apt install jq (Linux)"; exit 1; }
```

Do NOT proceed without `jq`. Do NOT fall back to other tools.

## Instructions

1. **Get the email.** The user's email is: `$ARGUMENTS`. If `$ARGUMENTS` is empty or missing, ask the user for their email address before proceeding.

2. **Validate the email.** The email must contain `@` followed by a domain with at least one dot. Reject any email containing characters outside of: letters, digits, `@`, `.`, `+`, `-`, `_`. If invalid, ask the user to provide a valid email. This allowlist prevents shell metacharacter injection.

3. **Register the user.** Define a reusable registration function that safely constructs JSON via `jq --arg`, calls the API, and parses the response. Never interpolate the email directly into shell strings:

```bash
register() {
  local TENANT_ID="$1"
  local EMAIL="$2"
  local RAW
  RAW=$(jq -n --arg tid "$TENANT_ID" --arg em "$EMAIL" \
    '{"tenant_id": $tid, "email": $em}' | \
    curl -sS -w '\n%{http_code}' -X POST https://memclaw.net/api/register \
      -H 'Content-Type: application/json' \
      --connect-timeout 10 --max-time 30 \
      -d @-)
  local EXIT_CODE=$?
  local HTTP_CODE
  HTTP_CODE=$(printf '%s\n' "$RAW" | tail -1)
  local BODY
  BODY=$(printf '%s\n' "$RAW" | sed '$d')
  echo "$EXIT_CODE"
  echo "$HTTP_CODE"
  echo "$BODY"
}
```

Derive a tenant_id from the email by replacing `@` with `-at-` and `.` with `-`, since the API does not allow `@` or `.` in tenant_id:

```bash
TENANT_ID=$(echo '<email>' | sed 's/@/-at-/g; s/\./-/g')
```

Call register and parse the three-part output:

```bash
OUTPUT=$(register "$TENANT_ID" '<email>')
CURL_EXIT=$(echo "$OUTPUT" | head -1)
HTTP_CODE=$(echo "$OUTPUT" | sed -n '2p')
BODY=$(echo "$OUTPUT" | tail -n +3)
```

Replace `<email>` with the actual email address.

4. **Handle errors:**
   - **Transport errors:** If `CURL_EXIT` is non-zero, curl failed at the network level (DNS, TLS, timeout). Show the user the error and suggest checking their network or https://memclaw.net status. Do not proceed.
   - **Tenant ID conflict (HTTP 409):** If the HTTP status is 409, or the body contains "already exists", "conflict", or "taken", generate a random suffix and retry using the same `register` function:
     ```bash
     for i in 1 2 3; do
       SUFFIX=$(openssl rand -hex 3 2>/dev/null || od -An -tx1 -N3 /dev/urandom | tr -d ' \n')
       OUTPUT=$(register "$TENANT_ID-$SUFFIX" '<email>')
       CURL_EXIT=$(echo "$OUTPUT" | head -1)
       HTTP_CODE=$(echo "$OUTPUT" | sed -n '2p')
       BODY=$(echo "$OUTPUT" | tail -n +3)
       if [ "$CURL_EXIT" = "0" ] && { [ "$HTTP_CODE" = "200" ] || [ "$HTTP_CODE" = "201" ]; }; then
         break
       fi
     done
     ```
     If all 3 retries fail, tell the user their email may already be registered and suggest they check https://memclaw.net.
   - **HTTP errors:** If the HTTP status code is not 200 or 201, tell the user the registration failed. Show both the HTTP status code and the response body. Suggest trying again later.
   - **Unexpected response:** If the body is not valid JSON or does not contain an `api_key` field, show the raw response body and ask the user to report the issue.

5. **Parse the response and extract the API key.** The success response looks like:
```json
{"tenant_id": "user-at-example-com", "api_key": "mc_...", "plan": "free"}
```
Extract the API key:
```bash
API_KEY=$(echo "$BODY" | jq -r '.api_key')
if [ -z "$API_KEY" ] || [ "$API_KEY" = "null" ]; then
  echo "ERROR: No api_key in response"
  echo "$BODY"
  exit 1
fi
```

6. **Store the API key.** Write the API key to the credentials file. Create the directory with restrictive permissions and check for errors on both write branches:

```bash
CREDS_FILE="$HOME/.claude/.credentials.json"

RESULT=$(
  umask 077
  mkdir -p "$(dirname "$CREDS_FILE")"
  if [ -f "$CREDS_FILE" ]; then
    if jq --arg key "$API_KEY" '.pluginConfigs["memclaw"].MEMCLAW_API_KEY = $key' "$CREDS_FILE" > "$CREDS_FILE.tmp" 2>/dev/null; then
      mv "$CREDS_FILE.tmp" "$CREDS_FILE" && echo "OK"
    else
      rm -f "$CREDS_FILE.tmp"
      echo "FAILED"
    fi
  else
    if jq -n --arg key "$API_KEY" '{"pluginConfigs": {"memclaw": {"MEMCLAW_API_KEY": $key}}}' > "$CREDS_FILE" 2>/dev/null; then
      echo "OK"
    else
      echo "FAILED"
    fi
  fi
)
```

If `$RESULT` is `FAILED`, tell the user the credential write failed, show them the path `~/.claude/.credentials.json`, and suggest they delete it and re-run the setup command.

Do NOT print the full API key in the chat — show only the first 8 characters followed by `...` so the user can identify it.

7. **Verify the connection.** Tell the user to run `/mcp` to confirm the memclaw server shows as connected. If it doesn't connect, suggest re-running `/memclaw:setup` with their email to refresh the credentials.

8. **Print a success message** with their tenant_id and plan (free). Remind them the command is `/memclaw:setup` if they need to re-run it.
