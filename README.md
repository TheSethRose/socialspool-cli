# @thesethrose/socialspool-cli

Agent-friendly CLI for the [SocialSpool](https://socialspool.com/) Public API.

SocialSpool is a social scheduling tool for creating posts, scheduling them
across connected publishing accounts, and checking final publish status. This
CLI is designed for humans and coding agents that need a safe, scriptable way
to work through the public API without using dashboard session routes.

## What You Can Do

- Inspect the authenticated workspace and API key scopes.
- List connected publishing accounts.
- Validate post content against account and platform constraints.
- Create drafts.
- Schedule posts for future publishing.
- Publish posts immediately.
- Wait for terminal publish status.
- Inspect timelines and failure details.
- Upload media assets when media support is enabled.
- Manage public API webhooks when the API key has webhook scopes.

## Install

```bash
npm install -g @thesethrose/socialspool-cli
```

Or run without a global install:

```bash
npx @thesethrose/socialspool-cli doctor --json
```

Requires Node.js 20.19 or newer.

## Authentication

Create a SocialSpool API key in the dashboard, then store it in your agent's
secure configuration: a profile `.env` file, config store, or secret manager.
Never write it to chat, logs, source control, or shell history.

Load it inline per invocation so it never lingers in the shell environment.

macOS / Linux:

```bash
# From a profile .env file
export $(grep -v '^#' ~/.hermes/profiles/<name>/.env | grep SOCIALSPOOL_API_KEY | xargs) && spool me --json

# Or inline for a single command
SOCIALSPOOL_API_KEY="ssp_live_..." spool me --json
```

Windows PowerShell:

```powershell
$env:SOCIALSPOOL_API_KEY="ssp_live_..."; spool me --json
```

For scheduling agents, use these scopes:

```text
posts:read
posts:write
accounts:read
```

Add `webhooks:read` and `webhooks:write` only when the agent should manage
webhooks.

The default API base URL is:

```text
https://socialspool.com/api/v1
```

Override it with:

```bash
export SOCIALSPOOL_API_BASE_URL=https://socialspool.com/api/v1
```

or pass `--base-url`.

Never paste API keys into prompts, logs, tickets, or generated files.

## Verify Setup

```bash
spool me --json
spool accounts list --json
spool capabilities --json
spool doctor --json
```

`doctor` checks authentication and core API reachability. If it fails, the JSON
output includes the structured error message and request id when the API
returned one.

## Package Update Notices

The CLI is primarily used by agents. When a newer npm package is available,
JSON command output may include a `package_update` object with the latest
version and update command. The notice is returned at most once per configured
API key and package version.

Agents should mention the update command when `package_update` is returned, but
must not run the update unless the user explicitly asks.

## Common commands

Complete command reference generated from the shared public action contracts:

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

List connected accounts:

```bash
spool accounts list --json
```

Validate content before scheduling:

```bash
spool posts validate \
  --content "Hello from SocialSpool" \
  --account acct_123 \
  --json
```

Create and schedule a post:

```bash
spool posts create \
  --content "Hello from SocialSpool" \
  --account acct_123 \
  --publish-at 2026-06-12T15:00:00.000Z \
  --idempotency-key schedule-launch-post-20260612 \
  --json
```

Wait for final status:

```bash
spool posts wait post_123 --json
```

Publish an existing draft now:

```bash
spool posts publish-now post_123 \
  --account acct_123 \
  --idempotency-key publish-now-post-123 \
  --json
```

Cancel a scheduled post:

```bash
spool posts cancel post_123 --json
```

Inspect a post timeline:

```bash
spool posts timeline post_123 --json
```

## Status Semantics

Do not treat a successful request as a successful platform publish.

- `draft`: saved but not scheduled.
- `scheduled`: accepted for future publishing.
- `publishing`: worker is attempting to publish.
- `published`: confirmed by the platform adapter.
- `failed`: terminal failure with an error code/message.
- `canceled`: scheduled publish was canceled.

Only report a post as published when SocialSpool returns `published` and
platform result data such as a platform post id or URL.

## Agent skill

This package includes `SKILL.md`, the SocialSpool Public API Agent skill. Give
that file to compatible coding agents along with an API key configured in their
runtime environment.

The skill intentionally excludes admin/customer-support operations. It uses the
public `spool` CLI only.

The future public install path is:

```bash
npx skills add socialspool/agent-skill
```

That external repository is not published yet. Repo-ready local files live in
`agent-skill/` in the SocialSpool source repo.

## Links

- Website: [https://socialspool.com/](https://socialspool.com/)
- API base URL: `https://socialspool.com/api/v1`
- OpenAPI contract: `https://socialspool.com/api/docs/openapi`
