# Market Kit Developer API and MCP Skill

Use this skill when you need to operate Market Kit programmatically through its developer surface.

This skill covers two related interfaces:

- The REST API served from Convex HTTP actions under `/api/v1`
- The remote Streamable HTTP MCP server served from `/mcp`

Both interfaces use the same bearer API key and the same underlying resource model.

## When to use this skill

Use it when you need to:

- Create, list, update, or delete brands
- Create, list, update, or delete campaigns
- Create, list, update, or delete posts
- Schedule or reschedule posts
- Start asset generation jobs and poll their status
- Create a brand draft from a website
- Remix an existing post into a new one
- Give another AI agent a reliable way to work with Market Kit over MCP

Do not use this skill for:

- Auth sign-in flows
- Billing, admin, or mail features
- Social account OAuth connection setup

Those are intentionally outside this developer surface.

## Mental model

The external contract uses stable public IDs, not raw Convex document IDs.

Common ID prefixes:

- `brand_...`
- `campaign_...`
- `post_...`
- `schedule_...`
- `job_...`
- `key_...`

The main hierarchy is:

1. A brand owns campaigns.
2. A campaign owns posts.
3. A post can have schedule history.
4. A campaign can also have asset generation jobs.

Important scheduling behavior:

- `POST /posts/{postId}/schedules` behaves like an external schedule collection API.
- Internally, it upserts the latest editable draft or scheduled record for that post.
- Listing schedules still returns history.

Important post update behavior:

- Updating a post can keep editable scheduled captions in sync.
- If a schedule already has a custom caption, Market Kit preserves that custom caption instead of overwriting it.

## Authentication

All requests use:

```http
Authorization: Bearer <API_KEY>
```

API keys are user-scoped. They only expose data owned by that user.

## REST API quick reference

Base URL:

```text
https://<your-convex-site>/api/v1
```

Key routes:

- `GET /brands`
- `POST /brands`
- `GET /brands/{brandId}`
- `PATCH /brands/{brandId}`
- `DELETE /brands/{brandId}`
- `POST /brands/from-website`
- `GET /brands/{brandId}/campaigns`
- `POST /brands/{brandId}/campaigns`
- `GET /brands/{brandId}/social-accounts`
- `GET /campaigns/{campaignId}`
- `PATCH /campaigns/{campaignId}`
- `DELETE /campaigns/{campaignId}`
- `GET /campaigns/{campaignId}/posts`
- `POST /campaigns/{campaignId}/posts`
- `GET /posts/{postId}`
- `PATCH /posts/{postId}`
- `DELETE /posts/{postId}`
- `POST /posts/{postId}/remix`
- `GET /schedules`
- `GET /posts/{postId}/schedules`
- `POST /posts/{postId}/schedules`
- `PATCH /schedules/{scheduleId}`
- `DELETE /schedules/{scheduleId}`
- `POST /campaigns/{campaignId}/asset-generation-jobs`
- `GET /asset-generation-jobs/{jobId}`
- `GET /api-keys`
- `POST /api-keys`
- `DELETE /api-keys/{keyId}`
- `GET /openapi.json`

## MCP quick reference

MCP transport:

- Streamable HTTP

MCP endpoint:

```text
https://<your-mcp-host>/mcp
```

MCP resources:

- `market-kit://openapi`
- `market-kit://docs/examples`

MCP tools:

- `list_brands`
- `create_brand`
- `update_brand`
- `delete_brand`
- `research_brand_from_website`
- `list_campaigns`
- `create_campaign`
- `update_campaign`
- `delete_campaign`
- `list_posts`
- `create_post`
- `update_post`
- `delete_post`
- `list_social_accounts`
- `list_schedules`
- `schedule_post`
- `reschedule_post`
- `cancel_schedule`
- `start_asset_generation`
- `get_asset_generation_job`
- `remix_post`

## Recommended workflow

When operating through this toolset, prefer this order:

1. List or create the brand.
2. Create or find the campaign.
3. Create or find the post.
4. If needed, list social accounts before scheduling.
5. Schedule the post.
6. If generating assets, start the job and poll until complete.

When you are uncertain about payload shape:

1. Read `market-kit://openapi` if using MCP.
2. Or fetch `/api/v1/openapi.json` if using REST.

## Easy examples

### Easy example 1: List brands over MCP

Goal: find what brands the current API key can access.

Use:

```json
tool: list_brands
args: {}
```

Expected outcome:

- A list of owned brands with public IDs like `brand_...`
- Use those IDs in later campaign and social-account calls

### Easy example 2: Create a brand over REST

```bash
curl -X POST "https://your-convex-site/api/v1/brands" \
  -H "Authorization: Bearer <YOUR_API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Northstar Coffee",
    "bio": "Single-origin coffee brand.",
    "brandVoice": "Warm, grounded, clear.",
    "brandDesignGuidelines": "Clean product framing, earthy tones, no heavy overlays.",
    "colors": ["#2F241F", "#D9C1A5"],
    "urls": ["https://northstar.example"]
  }'
```

Use this when you already know the brand data and do not need website research first.

### Easy example 3: Create a campaign and a manual post

First create the campaign:

```bash
curl -X POST "https://your-convex-site/api/v1/brands/brand_xxx/campaigns" \
  -H "Authorization: Bearer <YOUR_API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Spring Launch",
    "description": "Launch campaign for the seasonal range."
  }'
```

Then create the post:

```bash
curl -X POST "https://your-convex-site/api/v1/campaigns/campaign_xxx/posts" \
  -H "Authorization: Bearer <YOUR_API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Morning brew",
    "content": "Kick off the day with a smooth, chocolatey pour-over.",
    "platform": "instagram",
    "status": "Draft",
    "imageUrl": "https://cdn.example.com/posts/morning-brew.jpg",
    "imageRatio": "4:5",
    "tags": ["instagram", "coffee"]
  }'
```

## Complex examples

### Complex example 1: Full launch workflow over MCP

Goal: create a brand from a website, create a campaign, create a manual post, then schedule it.

Suggested sequence:

1. Research the brand:

```json
tool: research_brand_from_website
args: {
  "websiteUrl": "https://northstar.example"
}
```

2. Create a campaign under the returned `brand.id`:

```json
tool: create_campaign
args: {
  "brandId": "brand_123",
  "name": "Spring Launch",
  "description": "Launch campaign for the seasonal product line."
}
```

3. Create a post under the returned `campaign.id`:

```json
tool: create_post
args: {
  "campaignId": "campaign_123",
  "title": "Morning brew",
  "content": "Meet the new seasonal roast. Smooth body, cocoa finish, perfect for slow mornings.",
  "platform": "instagram",
  "imageUrl": "https://cdn.example.com/assets/spring-launch.jpg",
  "imageRatio": "4:5",
  "tags": ["launch", "seasonal", "coffee"]
}
```

4. List linked social accounts before scheduling:

```json
tool: list_social_accounts
args: {
  "brandId": "brand_123"
}
```

5. Schedule the post:

```json
tool: schedule_post
args: {
  "postId": "post_123",
  "scheduledAt": 1774252800000,
  "scheduledAtIso": "2026-03-22T10:00:00+00:00",
  "scheduledTimeZone": "Europe/London",
  "socialAccountIds": ["postforme_account_123"]
}
```

Use this flow when you want deterministic content creation and explicit review at each stage.

### Complex example 2: Start generation, poll, then schedule the chosen result

Goal: use the AI-assisted generation workflow and only schedule after the job is complete.

1. Start the asset generation job:

```json
tool: start_asset_generation
args: {
  "campaignId": "campaign_123",
  "topic": "Spring launch content for single-origin coffee",
  "platforms": ["instagram"],
  "selectedReferenceAssetUrls": [
    "https://cdn.example.com/references/brand-pack-hero.jpg"
  ],
  "productPageUrls": [
    "https://northstar.example/products/spring-roast"
  ]
}
```

2. Capture the returned `job.id`, then poll:

```json
tool: get_asset_generation_job
args: {
  "jobId": "job_123"
}
```

3. Repeat polling until the job reports completion.

4. List campaign posts:

```json
tool: list_posts
args: {
  "campaignId": "campaign_123"
}
```

5. Pick the generated post you want, then schedule it with `schedule_post`.

Use this flow when the creative asset should come from Market Kit's AI pipeline rather than a manually supplied image URL.

### Complex example 3: Safe update and reschedule flow

Goal: update post copy, preserve any custom schedule caption, and then move the publish time.

1. Update the post:

```json
tool: update_post
args: {
  "postId": "post_123",
  "content": "A brighter morning cup with citrus top notes and a rich cocoa finish."
}
```

2. Inspect schedule history:

```json
tool: list_schedules
args: {
  "from": 1774000000000,
  "to": 1775000000000
}
```

3. Reschedule the editable schedule:

```json
tool: reschedule_post
args: {
  "scheduleId": "schedule_123",
  "scheduledAt": 1774339200000,
  "scheduledAtIso": "2026-03-23T10:00:00+00:00",
  "scheduledTimeZone": "Europe/London"
}
```

Use this when you need to adjust copy and timing without recreating the post from scratch.

## Error handling guidance

If a request fails:

- Treat `401` as a missing, malformed, revoked, or invalid bearer API key.
- Treat `404` as a missing resource or a resource not owned by the current user.
- Treat `400` as an invalid payload or unsupported state transition.
- Read the JSON error message and use it verbatim when it is user-safe.

For AI agent behavior:

- Do not invent IDs. Always reuse real `brand_`, `campaign_`, `post_`, `schedule_`, and `job_` values returned by the API.
- Prefer list or fetch calls before mutation when context is ambiguous.
- Use OpenAPI or `market-kit://docs/examples` to recover exact request shapes.
- When scheduling, confirm that linked social accounts already exist. This surface does not perform the social OAuth linking flow.

## Best practices

- Prefer MCP when you want an AI agent to reason over tool outputs directly.
- Prefer REST when integrating with scripts, CI, or other backends.
- Use explicit website research only when a brand needs to be bootstrapped from a URL.
- Store API keys securely and treat the secret as write-capable user access.
- Revoke keys instead of reusing stale shared credentials.
