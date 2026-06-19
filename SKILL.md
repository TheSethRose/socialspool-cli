---
name: socialspool-public-api-agent
description: Use SocialSpool to inspect a workspace, list connected publishing accounts, create drafts, schedule posts, publish immediately, cancel scheduled posts, and verify post status through the public API and spool CLI.
version: 2.0.1
author: SocialSpool
metadata:
  hermes:
    tags: [socialspool, spool, cli, social-media, api, scheduling, agents]
---

# Skill: SocialSpool Public API Agent

## Purpose

Use SocialSpool to inspect a workspace, list connected publishing accounts, create drafts, schedule posts, publish immediately, cancel scheduled posts, and verify post status.

## When to use

Use this skill when a user asks an agent to create, schedule, publish, cancel, or inspect SocialSpool posts through the public API.

Use the hosted MCP connector instead of the CLI when the agent environment
supports remote MCP and OAuth:

```text
https://mcp.socialspool.com/mcp
```

Do not paste API keys into ChatGPT or other hosted remote MCP clients. The MCP
connector uses OAuth through the SocialSpool web session.

## Do not use this skill for

- Admin/customer-support operations, audit inspection, operations queues, or billing diagnostics (use `spool-admin` and `src/cli/spool-admin/SKILL.md` instead)
- User suspension/reactivation
- Workspace deletion
- Billing repair
- API key creation/revocation
- Direct database access
- Dashboard session-only routes
- Social OAuth connection flows
- Reading or handling social platform tokens
- Claiming platform publish success before SocialSpool reports confirmed platform result data

## Authentication

- Store the API key in your agent's secure configuration: a profile `.env` file, config store, or secret manager. Never write it to chat, logs, source control, or shell history.
- Load it inline per invocation so it never lingers in the shell environment.

macOS / Linux (bash/zsh):

```bash
# From a profile .env file
export $(grep -v '^#' ~/.hermes/profiles/<name>/.env | grep SOCIALSPOOL_API_KEY | xargs) && spool me --json

# Or inline for a single command
SOCIALSPOOL_API_KEY="ssp_live_..." spool me --json
```

Windows (PowerShell):

```powershell
$env:SOCIALSPOOL_API_KEY="ssp_live_..."; spool me --json
```

- Optional override: `SOCIALSPOOL_API_BASE_URL` or `--base-url`.
- If authentication fails, report the structured API error code and request ID if present.

## Required first checks

Every agent workflow must start with:

```bash
spool me --json
spool accounts list --json
spool capabilities --json
```

## Core workflow: schedule a post

1. Inspect workspace and scopes.
2. List connected accounts.
3. Validate content against selected account/platform constraints.
4. Create a draft or schedule directly.
5. Use an idempotency key for every write.
6. Verify returned post/target status.
7. Poll/wait until status is terminal when the user asked for confirmation.

Example:

```bash
spool accounts list --json

spool posts validate \
  --content "Post content here" \
  --account acct_123 \
  --publish-at 2026-06-12T15:00:00.000Z \
  --json

spool posts create \
  --content "Post content here" \
  --account acct_123 \
  --publish-at 2026-06-12T15:00:00.000Z \
  --idempotency-key schedule-<stable-operation-id> \
  --json

spool posts wait post_123 --json
```

## Core workflow: publish now

```bash
spool posts create --content "Post content here" --json

spool posts publish-now post_123 \
  --account acct_123 \
  --idempotency-key publish-now-<stable-operation-id> \
  --json

spool posts wait post_123 --json
```

## Core workflow: cancel scheduled post

```bash
spool posts cancel post_123 --json
spool posts get post_123 --json
```

## Core workflow: read analytics

Use analytics as proof-of-publish and a lightweight performance readout. Do not
claim analytics are complete when a platform reports delayed, unavailable, or
reconnect-required status.

```bash
spool analytics workspace --limit 20 --offset 0 --json
spool analytics post post_123 --json
```

## Media workflow

When attaching media, upload first, then reference returned media asset ids:

```bash
spool media config --json
spool media upload ./image.png --json
spool posts create --post-style text_media --content "Caption" --media-asset-ids media_123 --json
```

## Status semantics

- `draft`: not scheduled, not published.
- `scheduled`: accepted for future publishing.
- `publishing`: worker is attempting to publish.
- `published`: confirmed by platform adapter. Only this may be reported as published.
- `failed`: terminal failure. Report error code/message.
- `canceled`: no platform publish should occur.

## Hard rules

- Never claim publish success from a 200/202 response alone.
- Never claim success from `scheduled`, `queued`, or `publishing`.
- A post is published only when SocialSpool returns `published` plus platform result data such as platform post id or URL.
- If CLI JSON output includes `package_update`, mention the package update and its `update_command` in your response. Do not run the update command unless the user explicitly asks.
- Use idempotency keys for every POST/PATCH/DELETE.
- Retry only through public API/CLI commands.
- Do not call admin routes.
- Do not ask the user for social tokens.
- Do not use dashboard-only endpoints.
- Prefer JSON output.

## Troubleshooting

- Use `spool posts get <postId> --json`.
- Use `spool posts timeline <postId> --json`.
- For cross-workspace audit, billing, or operations investigation, use `spool-admin` instead of `spool`.

## Expected agent response

Return:

- command(s) run
- post id
- selected account ids/platforms
- idempotency key used
- final status
- platform URL/id when published
- error code/message when failed
- package update notice and update command when `package_update` is returned
- next required action if blocked

## CLI reference

<!-- PUBLIC_ACTIONS_CLI_REFERENCE:START -->
```bash
spool me
spool capabilities
spool accounts list
spool analytics workspace [--limit 20] [--offset 0]
spool analytics post <postId>
spool posts list [--status status] [--limit 20] [--offset 0]
spool posts get <postId>
spool posts timeline <postId>
spool posts validate --content <text> --account <id> [--publish-at ISO|now]
spool posts create [--content <text>] [--account <id>...] [--publish-at now|ISO]
spool posts update <postId> [--content <text>] [--title <title>] [--post-style style]
spool posts delete <postId> --yes
spool posts schedule <postId> --account <id>... --publish-at <ISO>
spool posts publish-now <postId> --account <id>...
spool posts cancel <postId>
spool media config
spool media delete <mediaAssetId>
spool webhooks list
spool webhooks create [--name <name>] --url <url> --event-type <type>|--event-types <a,b>
spool webhooks update <webhookId> [--name <name>] [--url <url>] [--event-types <a,b>] [--enabled true|false]
spool webhooks test <webhookId>
spool webhooks rotate-secret <webhookId>
spool webhooks delete <webhookId>
```
<!-- PUBLIC_ACTIONS_CLI_REFERENCE:END -->

## API base URL

Default production API:

```text
https://socialspool.com/api/v1
```

OpenAPI machine contract:

```text
https://socialspool.com/api/docs/openapi
```
