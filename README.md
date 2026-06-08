# Hostreach Agent Skills

Official Agent Skills for [Hostreach](https://hostreach.io) — the real-estate lead generation and CRM platform for Idealista and other portals.

## Install

### One command (no CLI needed)

```bash
# Interactive — choose agent and scope
npx skills add hostreach/agent-skills

# Global install for Cursor
npx skills add hostreach/agent-skills -g -a cursor

# Global install for Claude Code
npx skills add hostreach/agent-skills -g -a claude-code

# Global install for all supported agents
npx skills add hostreach/agent-skills -g --all
```

> Uses [vercel-labs/skills](https://github.com/vercel-labs/skills) under the hood. No global install required.

### Via Hostreach CLI

```bash
npm install -g @hostreach/cli

hostreach skill install                    # Cursor (default), global
hostreach skill install --agent claude     # Claude Code, global
hostreach skill install --all              # all agents, global
hostreach skill install --all --project    # all agents, project-level
```

## Available Skills

| Skill | Description |
|-------|-------------|
| `hostreach` | Extract leads from Idealista and portals, manage campaigns, CRM pipeline, WhatsApp messaging, valuations, fichas, and scheduling. |

## What the skill teaches agents

- When to use MCP server vs CLI vs direct HTTP API
- Authentication methods and priority order
- All CLI commands with flags and examples
- Lead statuses and valid enum values
- How to poll async extraction jobs safely
- Pagination patterns
- Error codes and recovery actions
- Scopes and what a `HTTP_403` means
- Common workflows: lead extraction, campaigns, CRM, valuations, fichas, webhooks

## Keeping skills up to date

```bash
npx skills update hostreach/agent-skills
```

## Also available

- **MCP server** — for Cursor, Claude Desktop, and Codex:
  ```bash
  hostreach agent add cursor
  ```
- **CLI** — 100+ endpoints, all output formats:
  ```bash
  npm install -g @hostreach/cli
  hostreach list
  ```

## Links

- [Hostreach dashboard](https://app.hostreach.io)
- [API docs](https://docs.hostreach.io)
- [npm package `@hostreach/cli`](https://www.npmjs.com/package/@hostreach/cli)
- [OpenAPI spec](https://api.hostreach.io/v1/public/v1/openapi.json)
