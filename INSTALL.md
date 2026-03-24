# Install and Setup Guide

This guide explains how to get the Market Kit developer surface running for an AI agent or any MCP-capable client.

It covers:

- REST API setup
- API key creation
- Local MCP server setup
- Client configuration
- First-run verification
- Mail REST and MCP capabilities

## What you are installing

Market Kit exposes two connected pieces:

1. A Convex-backed REST API under `/api/v1`
2. A remote Streamable HTTP MCP server under `/mcp`

The MCP server is a thin adapter over the REST API. It uses the same bearer API key and exposes Market Kit actions as MCP tools and resources.

Mail is included in that same surface, including mailbox management, folder listing, inbox-style email listing, stable system-folder filtering, search, per-thread drafts, AI reply drafting, reply flows, and sending new outbound email.

## Prerequisites

Before you start, make sure you have:

- Node.js 20+
- npm
- A running Convex deployment for this app
- A deployed or local frontend if you want to create API keys through the UI
- A reachable Convex site URL for HTTP actions

## Step 1: Install project dependencies

From the repo root:

```bash
npm install
```

## Step 2: Configure the app and Convex

Frontend local env:

```env
VITE_CONVEX_URL=https://your-deployment.convex.cloud
```

Convex env values:

```bash
npx convex env set CONVEX_SITE_URL https://your-app.example.com
npx convex env set MARKET_KIT_API_BASE_URL https://your-deployment.convex.site
npx convex env set MARKET_KIT_MCP_SERVER_URL https://your-mcp-host.example.com/mcp
```

Notes:

- `MARKET_KIT_API_BASE_URL` must point at the Convex HTTP-action host, not the frontend host.
- The REST API lives under `https://your-deployment.convex.site/api/v1`.
- `MARKET_KIT_MCP_SERVER_URL` is what the in-app Developers page shows in copyable MCP config examples.

If you are running everything locally, a practical setup looks like this:

```bash
npx convex env set CONVEX_SITE_URL http://localhost:5173
npx convex env set MARKET_KIT_API_BASE_URL http://127.0.0.1:3210
npx convex env set MARKET_KIT_MCP_SERVER_URL http://localhost:3337/mcp
```

Use the actual local Convex site URL available in your environment.

## Step 3: Deploy Convex functions and schema

Push the backend so the developer API tables, indexes, and functions exist:

```bash
npx convex deploy -y --typecheck disable
```

If you are working in local development instead of production:

```bash
npx convex dev
```

Important:

- The `/developers` page and `/api/v1` routes depend on the `apiKeys` table and new public-ID indexes.
- If the frontend is newer than the Convex deployment, the Developers page can fail until Convex is deployed too.

## Step 4: Create an API key

Recommended path:

1. Open the Market Kit app.
2. Go to the `Developers` page.
3. Create a new API key.
4. Copy the secret immediately.

The secret is only shown once.

Alternative REST path:

If you already have a working API key and need to mint another one:

```bash
curl -X POST "https://your-convex-site/api/v1/api-keys" \
  -H "Authorization: Bearer <EXISTING_API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Local MCP client"
  }'
```

## Step 5: Start the local MCP server

From the repo root:

```bash
npm run dev:mcp
```

This starts the server defined in:

- `services/market-kit-mcp/server.ts`

The server expects one of these env vars:

- `MARKET_KIT_API_BASE_URL`
- `CONVEX_SITE_URL`

The MCP server defaults to:

- Host: `0.0.0.0`
- Port: `3337`
- Path: `/mcp`

Health check:

```bash
curl "http://localhost:3337/health"
```

Expected response shape:

```json
{
  "apiBaseUrl": "https://your-deployment.convex.site",
  "mcpServerUrl": "http://localhost:3337/mcp",
  "ok": true
}
```

## Step 6: Connect your AI client

Use Streamable HTTP MCP with a bearer API key header.

Example config:

```json
{
  "mcpServers": {
    "marketKit": {
      "type": "streamable-http",
      "url": "http://localhost:3337/mcp",
      "headers": {
        "Authorization": "Bearer <YOUR_API_KEY>"
      }
    }
  }
}
```

Hosted example:

```json
{
  "mcpServers": {
    "marketKit": {
      "type": "streamable-http",
      "url": "https://your-mcp-host.example.com/mcp",
      "headers": {
        "Authorization": "Bearer <YOUR_API_KEY>"
      }
    }
  }
}
```

## Step 7: First-run verification

Verify the REST API first:

```bash
curl "https://your-convex-site/api/v1/openapi.json" \
  -H "Authorization: Bearer <YOUR_API_KEY>"
```

Then verify the MCP server through your client by checking:

- Resource: `market-kit://openapi`
- Resource: `market-kit://docs/examples`
- Tool: `list_brands`
- Tool: `list_emails`

If those work, the install is good.

## Fast start examples

### Fast start with REST

Create a brand:

```bash
curl -X POST "https://your-convex-site/api/v1/brands" \
  -H "Authorization: Bearer <YOUR_API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Northstar Coffee",
    "bio": "Single-origin coffee brand.",
    "colors": ["#2F241F", "#D9C1A5"],
    "urls": ["https://northstar.example"]
  }'
```

### Fast start with MCP

Call:

```json
tool: list_brands
args: {}
```

Then create a campaign:

```json
tool: create_campaign
args: {
  "brandId": "brand_123",
  "name": "Spring Launch",
  "description": "Launch content for the seasonal range."
}
```

### Fast start with Mail over REST

List mailbox emails that are waiting for a reply from the Inbox system folder:

```bash
curl "https://your-convex-site/api/v1/mailboxes/mailbox_xxx/emails?view=waiting_for_reply&folder=inbox&limit=25" \
  -H "Authorization: Bearer <YOUR_API_KEY>"
```

List all mailbox emails with the stable `all` folder selector:

```bash
curl "https://your-convex-site/api/v1/mailboxes/mailbox_xxx/emails?folder=all&view=all&limit=50" \
  -H "Authorization: Bearer <YOUR_API_KEY>"
```

Search mailbox emails:

```bash
curl -X POST "https://your-convex-site/api/v1/mailboxes/mailbox_xxx/emails/search" \
  -H "Authorization: Bearer <YOUR_API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{
    "folder": "all",
    "q": "order 1042",
    "limit": 25
  }'
```

Save a draft for a thread:

```bash
curl -X PUT "https://your-convex-site/api/v1/mailboxes/mailbox_xxx/threads/thread_xxx/draft" \
  -H "Authorization: Bearer <YOUR_API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{
    "mode": "replyAll",
    "to": ["customer@example.com"],
    "cc": ["support@example.com"],
    "subject": "Re: Order #1042",
    "body": "Thanks for the update. We have checked the order and will follow up shortly.",
    "attachments": []
  }'
```

### Fast start with Mail over MCP

List triaged emails:

```json
tool: list_emails
args: {
  "mailboxId": "mailbox_123",
  "folder": "inbox",
  "view": "waiting_for_reply",
  "limit": 25
}
```

Search emails:

```json
tool: search_emails
args: {
  "mailboxId": "mailbox_123",
  "folder": "all",
  "q": "order 1042",
  "limit": 25
}
```

Generate an AI reply draft:

```json
tool: generate_ai_mail_reply
args: {
  "mailboxId": "mailbox_123",
  "threadId": "thread_123",
  "mode": "replyAll"
}
```

## Complex onboarding example

This is a good full-system smoke test for a new AI client:

1. Call `research_brand_from_website` with a real brand URL.
2. Call `create_campaign` using the returned `brand.id`.
3. Call `create_post` using the returned `campaign.id`.
4. Call `list_social_accounts` for the brand.
5. Call `schedule_post` with one of the returned social account IDs.
6. Call `list_schedules` to verify the scheduled item exists.

If you prefer AI-generated content instead of a manual post:

1. Call `start_asset_generation`.
2. Poll with `get_asset_generation_job`.
3. When the job completes, call `list_posts`.
4. Schedule one of the generated posts.

## Mail capability checklist

Current Mail REST routes:

- `GET /brands/{brandId}/mailboxes`
- `POST /brands/{brandId}/mailboxes/discover`
- `POST /brands/{brandId}/mailboxes/test`
- `POST /brands/{brandId}/mailboxes`
- `GET /mailboxes/{mailboxId}`
- `PATCH /mailboxes/{mailboxId}`
- `DELETE /mailboxes/{mailboxId}`
- `POST /mailboxes/{mailboxId}/attachments/upload-url`
- `GET /mailboxes/{mailboxId}/folders`
- `GET /mailboxes/{mailboxId}/emails`
- `POST /mailboxes/{mailboxId}/emails/search`
- `GET /mailboxes/{mailboxId}/emails/{threadId}`
- `POST /mailboxes/{mailboxId}/emails/{threadId}/reply`
- `POST /mailboxes/{mailboxId}/emails/send`
- `GET /mailboxes/{mailboxId}/threads`
- `GET /mailboxes/{mailboxId}/threads/{threadId}`
- `GET /mailboxes/{mailboxId}/threads/{threadId}/draft`
- `PUT /mailboxes/{mailboxId}/threads/{threadId}/draft`
- `DELETE /mailboxes/{mailboxId}/threads/{threadId}/draft`
- `POST /mailboxes/{mailboxId}/threads/{threadId}/ai-reply`
- `POST /mailboxes/{mailboxId}/threads/{threadId}/send-reply`

Mail email-list filters:

- `view=all`
- `view=unread`
- `view=new`
- `view=waiting_for_reply`
- `view=needs_reply`
- `folder=all`
- `folder=inbox`
- `folder=sent`
- `folder=archive`
- `folder=spam`
- `folder=drafts`
- optional `q`
- optional `folderPath`
- optional `limit`

Recommended usage:

- Prefer `/emails` for the primary inbox workflow.
- Prefer `folder` for stable cross-provider mailbox views.
- Use raw `folderPath` only when you need a provider-specific IMAP path.
- `view=all&folder=all` is the broadest inbox listing shape.

## Workflow examples worth checking first

The live OpenAPI and MCP docs examples now include workflow sequences for:

- create brand → create campaign → create post → generate asset → schedule
- list mailbox → list threads → generate draft → send reply
- build social content from website
- regenerate failed asset
- reschedule a queued post

You can inspect those through:

- REST: `/api/v1/openapi.json`
- MCP resource: `market-kit://docs/examples`

## Common problems

### The Developers page throws a server error

Likely cause:

- The frontend is deployed, but Convex schema and functions were not deployed.

Fix:

```bash
npx convex deploy -y --typecheck disable
```

### MCP returns 401

Likely causes:

- Missing `Authorization` header
- Header is not in `Bearer <token>` format
- API key is revoked or invalid

### MCP server will not start

Likely cause:

- `MARKET_KIT_API_BASE_URL` and `CONVEX_SITE_URL` are both missing

Fix:

Set one of them before running:

```bash
set MARKET_KIT_API_BASE_URL=https://your-deployment.convex.site
npm run dev:mcp
```

### Scheduling fails

Likely cause:

- No linked social accounts exist yet for that brand

Fix:

- Link the social account through the main app first
- Then retry `list_social_accounts` and `schedule_post`

## What an AI agent should do first

When an AI connects for the first time, the safest startup flow is:

1. Read `market-kit://openapi`
2. Read `market-kit://docs/examples`
3. Call `list_brands`
4. If no brands exist, decide whether to:
   - use `create_brand` with known fields
   - or use `research_brand_from_website`

That gives the agent the schema, working examples, and current state before it makes changes.
