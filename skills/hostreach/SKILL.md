---
name: hostreach
description: Use when interacting with Hostreach — extract real-estate leads from Idealista and other portals, manage campaigns, CRM lead pipeline, WhatsApp messaging, property valuations, fichas, and scheduling. Provides CLI, MCP server and direct API access.
---

# Hostreach Agent Skill

Use this skill when the user asks you to interact with Hostreach: extract leads, manage campaigns, view inbox, create valuations, generate fichas, schedule appointments, or perform any action available on the platform.

## When to use MCP vs CLI vs direct API

| Situation | Recommended |
|-----------|-------------|
| Conversational agent in Cursor / Claude Desktop | **MCP server** — structured tools, no shell needed |
| Scripts, CI/CD, pipes, batch jobs | **CLI** — composable, stdout/stderr split, all output formats |
| Custom integrations without Node.js | **Direct API** — HTTP + `X-Api-Key` header, OpenAPI spec available |
| Need output as file/CSV for analysis | **CLI** with `--output` or `--format csv` |

## Install

```bash
# Install the CLI globally (includes MCP server + Agent Skill)
npm install -g @hostreach/cli

# Or run without installing
npx -p @hostreach/cli hostreach campaigns list --api-key hr_live_...
```

## Authentication

Three ways to authenticate, in priority order:

| Priority | Method | Example |
|----------|--------|---------|
| 1 | `--api-key` flag | `hostreach campaigns list --api-key hr_live_...` |
| 2 | Config file | `hostreach auth login` (saves to `~/.config/hostreach/`) |
| 3 | Environment variable | `export HOSTREACH_API_KEY=hr_live_...` |

Generate an API key from **Settings › Workspace › Acceso API** in the Hostreach dashboard.

```bash
hostreach auth login      # interactive — saves key + base URL
hostreach auth status     # show current auth
hostreach auth logout     # remove stored credentials
```

## CLI Usage

Every API endpoint is a subcommand under its resource:

```bash
hostreach <resource> <action> [--flags]
```

### Discover endpoints

```bash
hostreach list                     # list all resources
hostreach list campaigns           # list actions for campaigns
hostreach campaigns list --help    # show parameters for an action
```

### Examples

```bash
# Leads
hostreach leads list --status PENDING --limit 20
hostreach leads get --id <leadId>
hostreach leads update --id <leadId> --status CONTACTED

# Campaigns
hostreach campaigns list
hostreach campaigns create --platform idealista --name "Valencia Rent Q3"
hostreach campaigns create --platform csv_import --name "CSV Drip" --segmentation '{"batchSize":10}' --settings '{"scheduledHours":"9,14","scheduledDays":"1,2,3,4,5"}'
hostreach campaigns create --platform widget --name "Widget inbound"
hostreach campaigns update-status --id <campaignId> --status running
hostreach campaigns execute --id <campaignId>
hostreach campaigns stats --id <campaignId>

# Extractions (portal platforms only — one-shot async scrape)
hostreach extractions create --platform idealista --filters '{"operation":"rent","location":{"id":"0-EU-ES-VC","label":"Valencia"}}'
hostreach extractions get --id <extractionId>
hostreach extractions list --page 1 --limit 20

# Platforms
hostreach platforms list
hostreach platforms filters --platform idealista
hostreach platforms locations --platform idealista --q "Valencia"

# Messaging
hostreach messaging conversations --limit 20
hostreach messaging send --lead-id <id> --message "Hola!"

# Valuations
hostreach valuations create --name "Piso Gran Via 45" --platform idealista
hostreach valuations list

# Fichas
hostreach fichas generate --idealista-url "https://www.idealista.com/inmueble/12345678/"
hostreach fichas list

# Scheduling
hostreach scheduling list-event-types
hostreach scheduling availability --event-type-id <id>
hostreach scheduling appointments
```

### Output formats

```bash
hostreach campaigns list --pretty                      # pretty-print JSON
hostreach leads list --format table                    # ASCII table
hostreach leads list --format csv > leads.csv          # CSV export
hostreach leads list --format markdown                 # Markdown table
hostreach campaigns list --output ./data.json          # save to file, print path
hostreach leads list --clean                           # strip nulls/empty values
```

All status messages go to **stderr**. Data goes to **stdout** — safe for piping.

Structured errors for agents:
```json
{ "error": true, "code": "HTTP_401", "message": "...", "suggestion": "Run 'hostreach auth login'" }
```

### Refresh cached spec

```bash
hostreach campaigns list --refresh    # force re-fetch of OpenAPI spec
```

## MCP Server (recommended for Cursor / Claude Desktop)

### Auto-configure (easiest)

```bash
hostreach agent add cursor    # writes .cursor/mcp.json in project
hostreach agent add claude    # writes ~/.claude/claude_desktop_config.json
hostreach agent add codex     # writes ~/.codex/mcp.json
```

Merges into existing config without overwriting other MCP servers. Prompts for API key if not already stored.

### Manual configuration

Add to `~/.cursor/mcp.json` or `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "hostreach": {
      "command": "npx",
      "args": ["-y", "--package=@hostreach/cli", "hostreach-mcp"],
      "env": {
        "HOSTREACH_API_KEY": "hr_live_your_key_here"
      }
    }
  }
}
```

Available MCP tools:

**Identity & usage:** `get_me`, `get_usage`

**Platforms:** `list_platforms`, `get_platform_filters`, `search_locations`

**Extractions:** `start_extraction`, `get_extraction`, `list_extractions`, `get_extraction_leads`

**Campaigns:** `list_campaigns`, `get_campaign`, `create_campaign`, `update_campaign_status`, `execute_campaign`, `get_campaign_stats`

**Leads:** `list_leads`, `get_lead`, `update_lead_status`, `add_lead_note`, `get_lead_notes`

**Messaging:** `send_message`, `send_template`, `get_conversations`

**Webhooks:** `list_webhooks`, `create_webhook`, `update_webhook`, `delete_webhook`, `test_webhook`

**Templates:** `list_templates`, `create_template`, `update_template`, `delete_template`

**Accounts (WhatsApp):** `list_accounts`, `create_account`, `delete_account`

**Valuations:** `list_valuations`, `create_valuation`, `get_valuation`, `get_valuation_comparables`, `delete_valuation`

**Fichas:** `list_fichas`, `generate_ficha`, `get_ficha`, `delete_ficha`

**Scheduling:** `list_event_types`, `create_event_type`, `update_event_type`, `get_event_type`, `get_availability`, `list_appointments`, `cancel_appointment`

## Agent Skill

Install the skill to give AI agents deep knowledge of Hostreach APIs:

```bash
# Install for a specific agent
hostreach skill install --agent cursor      # default
hostreach skill install --agent claude
hostreach skill install --agent codex
hostreach skill install --agent opencode

# Install for all supported agents at once
hostreach skill install --all

# Project-level (cwd) instead of global (~/)
hostreach skill install --agent cursor --project
hostreach skill install --all --project

# Dump skill to stdout (paste anywhere)
hostreach skill print
```

Or install directly from GitHub without the CLI:

```bash
npx skills add hostreach/agent-skills                         # interactive
npx skills add hostreach/agent-skills -g -a cursor            # global, Cursor only
npx skills add hostreach/agent-skills -g --all                # global, all agents
```

## Direct API (without CLI or MCP)

Base URL: `https://api.hostreach.io/v1/public/v1`

Authenticate: `X-Api-Key: hr_live_...` or `Authorization: Bearer hr_live_...`

OpenAPI spec: `https://api.hostreach.io/v1/public/v1/openapi.json`

## Lead statuses

Always use one of these exact values when filtering or updating a lead's status:

| Status | Meaning |
|--------|---------|
| `PENDING` | Newly extracted, not yet contacted (system-set; cannot be assigned manually) |
| `CONTACTED` | Outreach message sent |
| `REPLIED` | Lead responded |
| `QUALIFIED` | Confirmed interest |
| `UNQUALIFIED` | Not a fit |
| `MEETING_SCHEDULED` | Appointment booked |
| `FOLLOWED_UP` | Follow-up sent |
| `NEGOTIATION` | In active negotiation |
| `WON` | Deal closed |
| `LOST` | Deal lost |
| `ARCHIVED` | Hidden from active pipeline |
| `ERROR` | Delivery error |

## Campaign types

The `platform` field is the discriminator that determines how leads enter a campaign.

| Platform | Type | Leads source | Modes available | Notes |
|----------|------|-------------|-----------------|-------|
| `idealista` | Portal | Scheduled/manual scrape from Idealista (Spain) | `extract_only`, `extract_and_contact` | Use `GET /platforms/idealista/filters` |
| `fotocasa` | Portal | Scheduled/manual scrape from Fotocasa (Spain) | `extract_only`, `extract_and_contact` | |
| `metrocuadrado` | Portal | Scrape from Metrocuadrado (Colombia) | `extract_only`, `extract_and_contact` | |
| `inmuebles24` | Portal | Scrape from Inmuebles24 (Mexico) | `extract_only`, `extract_and_contact` | |
| `zonaprop` | Portal | Scrape from Zonaprop (Argentina) | `extract_only`, `extract_and_contact` | |
| `csv_import` | CSV | Leads uploaded via dashboard CSV | `extract_and_contact` only (forced) | Requires `batchSize` + schedule |
| `widget` | Widget | Inbound from embedded Hostreach widget form | `extract_and_contact` only (forced) | Max 1 running per workspace |

### Portal campaign — key params

```json
{
  "platform": "idealista",
  "name": "Valencia Rent Q3 2026",
  "mode": "extract_and_contact",
  "contactMode": "whatsapp",
  "segmentation": {
    "filters": { "operation": "rent", "location": { "id": "0-EU-ES-VC-46", "label": "Valencia" } },
    "expectedResults": 50
  }
}
```

The campaign is created as `draft`. Set it to `running` via `PATCH /:id/status` or use `POST /:id/execute` for a one-off manual run.

### CSV import campaign — key params

```json
{
  "platform": "csv_import",
  "name": "My CSV drip",
  "contactMode": "whatsapp",
  "segmentation": { "batchSize": 10 },
  "settings": { "scheduledHours": "9,14:30", "scheduledDays": "1,2,3,4,5" }
}
```

`batchSize` (1–25) = leads contacted per scheduled run. `scheduledHours` and `scheduledDays` are required before transitioning to `running`. Upload leads via the dashboard.

### Widget campaign — key params

```json
{
  "platform": "widget",
  "name": "Website inbound",
  "contactMode": "whatsapp"
}
```

No segmentation or manual execution. Only one `running` widget campaign per workspace at a time.

### Campaign statuses

| Status | Meaning |
|--------|---------|
| `draft` | Created, not yet active. Configure and set to `running` when ready. |
| `running` | Active — scheduler runs on `scheduledHours`/`scheduledDays`. |
| `paused` | Suspended — no new extractions or messages, resumes when set back to `running`. |
| `completed` | Ended (CSV campaigns finish after all leads are processed). |

## Async extractions / polling

Extractions are asynchronous jobs. After `start_extraction` / `extractions create`, poll until complete:

```bash
# CLI polling pattern (use in a loop with sleep)
hostreach extractions get --id <id>    # check status field

# Status values: QUEUED | RUNNING | COMPLETED | FAILED
# Poll every 5–10s; stop after COMPLETED or FAILED (do not loop forever)
```

Via MCP, call `get_extraction` and check `status`. If `status === 'FAILED'`, surface `error` field to the user.

**Note:** Extractions only support portal platforms (`idealista`, `fotocasa`, `metrocuadrado`, `inmuebles24`, `zonaprop`). For `widget` or `csv_import`, create a campaign via `create_campaign` / `campaigns create` instead.

## Pagination

Resources with lists support `--page` (1-based) and `--limit` (default 20, max 100):

```bash
hostreach leads list --page 1 --limit 50
hostreach leads list --page 2 --limit 50
```

Stop when the returned array has fewer items than `--limit` (last page reached). Do not assume a fixed total unless the response includes a `total` field.

## Error handling for agents

Structured errors are always printed to **stdout** as JSON with `"error": true`:

```json
{ "error": true, "code": "HTTP_403", "message": "This API key does not have the required scope: campaigns:write", "suggestion": "Create a new API key with the campaigns:write scope." }
```

| Code | Cause | Action |
|------|-------|--------|
| `HTTP_401` | Invalid or missing API key | Run `hostreach auth login` or pass `--api-key` |
| `HTTP_403` | Key active but scope insufficient | Create/rotate key with the required scope |
| `HTTP_404` | Resource not found | Verify the ID; resource may have been deleted |
| `HTTP_429` | Rate or quota limit reached | Check usage with `get_usage`; wait or upgrade |
| `HTTP_500` | Server error | Retry once; surface message to user if persists |

## Common Workflows

### Extract leads from Idealista

```
1. hostreach auth status — confirm workspace and quota
2. hostreach platforms locations --platform idealista --q "Valencia"
3. hostreach extractions create --platform idealista --filters '{"location":<result>,"operation":"rent"}' --max-results 100
4. hostreach extractions get --id <id>   — poll every 5s until status=COMPLETED
5. hostreach extractions leads --id <id>
```

### Manage campaign lifecycle (portal)

```
1. hostreach campaigns list
2. hostreach platforms locations --platform idealista --q "Valencia"
3. hostreach campaigns create --platform idealista --name "Valencia Q3" --segmentation '{"filters":{"operation":"rent","location":<result>},"expectedResults":50}'
4. hostreach campaigns update-status --id <id> --status running
5. hostreach campaigns execute --id <id>   # manual one-off run
6. hostreach campaigns stats --id <id>
```

### Create and activate a CSV drip campaign

```
1. hostreach campaigns create --platform csv_import --name "Contacts batch" --segmentation '{"batchSize":10}' --settings '{"scheduledHours":"9,14","scheduledDays":"1,2,3,4,5"}'
2. # Upload leads via dashboard (CSV import)
3. hostreach campaigns update-status --id <id> --status running
```

### CRM pipeline update

```
1. hostreach leads list --status PENDING --limit 20
2. hostreach leads update --id <id> --status CONTACTED
3. hostreach leads notes create --id <id> --content "Cliente interesado, llamar martes 10h"
4. hostreach messaging send --lead-id <id> --message "Hola! Vi tu propiedad..."
```

### Generate a property valuation

```
1. hostreach platforms list
2. hostreach platforms locations --platform idealista --q "Madrid Salamanca"
3. hostreach valuations create --name "Apto Gran Via 45" --platform idealista --filters '{"location":<result>}'
4. hostreach valuations get --id <id>
5. hostreach valuations comparables --id <id>
```

### Generate a property ficha from Idealista

```
1. hostreach fichas generate --idealista-url "https://www.idealista.com/inmueble/12345678/"
2. hostreach fichas get --id <id>
```

### Set up a webhook for lead events

```
1. hostreach webhooks create --url "https://my-app.com/hooks" --events '["leads.imported","lead.status_updated"]'
2. hostreach webhooks test --id <id>
3. hostreach webhooks list
```

### Send a WhatsApp message / template

```bash
# Plain-text message
hostreach messaging send --lead-id <leadId> --message "Hola, ¿sigue disponible el piso?"

# Template message (scheduled)
hostreach messaging send-template --lead-id <leadId> --template-id <templateId> --scheduled-at "2026-06-10T10:00:00Z"
```

MCP:
```
send_message(leadId, message, accountId?, campaignId?)
send_template(leadId, templateId, accountId?, scheduledAt?, campaignId?)
```

### Create a property valuation

MCP:
```json
create_valuation({
  "latitude": 40.4168, "longitude": -3.7038,
  "squareMeters": 80, "bedrooms": 3, "bathrooms": 2,
  "propertyType": "homes", "operation": "sale",
  "name": "Calle Mayor 12, 3º B"
})
```

### Connect a WhatsApp account

**Meta Cloud API:**
```json
POST /accounts
{
  "type": "whatsapp",
  "whatsAppBusinessAccountId": "123456789012345",
  "token": "EAAG...",
  "phone": "+34600000000"
}
```

**Evolution API:**
```json
POST /accounts
{
  "type": "whatsapp-no-api",
  "phone": "+34600000000",
  "evolutionInstanceId": "instance-1",
  "evolutionApiUrl": "https://evo.example.com",
  "evolutionApiKey": "your-key"
}
```

### Create a WhatsApp template

```json
POST /templates
{
  "apiType": "meta-api",
  "platform": "idealista",
  "wabaId": "123456789012345",
  "name": "bienvenida_inquilinos_v2",
  "category": "MARKETING",
  "language": "es",
  "components": [
    { "type": "BODY", "text": "Hola {{1}}, vi tu anuncio en Idealista. ¿Sigues buscando piso?" }
  ]
}
```

### Create a scheduling event type

```json
POST /scheduling/event-types
{
  "ownerAgentUserId": "<agentUserId>",
  "name": "Property Tour",
  "slug": "property-tour",
  "durationMin": 30,
  "locationMode": "onsite"
}
```

### Create a webhook subscription

```json
POST /webhooks
{
  "url": "https://yourapp.com/webhooks/hostreach",
  "secret": "your-signing-secret",
  "name": "Production webhook",
  "events": ["leads.imported", "lead.status_updated", "lead.first_response"]
}
```

Supported events: `extraction.completed`, `leads.imported`, `lead.status_updated`, `lead.first_response`.

## Scopes

When creating a limited API key, choose the scopes matching the actions you need. A `HTTP_403` response means the active key is missing the required scope — create a new key with that scope added.

| Scope | Grants |
|-------|--------|
| `campaigns:read/write` | List, create, update campaigns |
| `leads:read/write` | List, update, delete leads and notes |
| `extractions:read/write` | Start and inspect extraction jobs |
| `platforms:read` | Platform filter schemas and location search |
| `messaging:read/write` | Read/send WhatsApp messages |
| `templates:read/write` | Message templates |
| `accounts:read/write` | WhatsApp accounts |
| `valuations:read/write` | Property valuations |
| `fichas:read/write` | Listing sheets |
| `scheduling:read/write` | Event types and appointments |
| `webhooks:read/write` | Webhook subscriptions |

An empty `scopes` array = full access (wildcard).
