---
name: memclaw-memory
description: Persistent AI memory via MemClaw — store and retrieve context across sessions
---

# MemClaw Memory Skill

MemClaw provides persistent memory across Claude Code sessions via MCP tools.

## When to Store Memories

Proactively store memories when:
- The user makes an **architectural decision** (e.g. "we're using PostgreSQL", "monorepo structure")
- **Tech stack choices** are finalized (frameworks, libraries, versions)
- A **debugging gotcha** is discovered (e.g. "this API requires trailing slash")
- The user states a **preference** (coding style, tooling, workflow)
- **Project conventions** are established (naming, file structure, patterns)
- Important **context** would be lost between sessions

## When to Retrieve Memories

Proactively retrieve memories when:
- **Starting a new session** in a project — check for prior context
- **Entering an unfamiliar area** of the codebase — check if past decisions apply
- **Before making architectural decisions** — check if prior decisions constrain the choice
- The user asks **"what did we decide about X"** or similar recall questions
- **Onboarding to a project** — pull all stored context for orientation

## Troubleshooting

- If the memclaw MCP server isn't connected, tell the user to run `/memclaw:memclaw-setup`
- If a tool call returns an authentication or 401 error, the API key may be missing or invalid — first suggest re-running `/memclaw:memclaw-setup` with their email to refresh the credentials. If that doesn't help, direct them to `/plugin` > memclaw > configure to manually re-enter the key
- If a tool call fails with a network error, suggest the user check https://memclaw.net for service status

## Usage Notes

- Memories are scoped by tenant — each user has their own memory space
- Store memories with clear, searchable descriptions
- Prefer storing decisions and rationale, not raw code snippets
- Keep stored memories concise and actionable
