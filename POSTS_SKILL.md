# Market Kit Posts Skill

Use this skill when you need to operate Market Kit's content side through REST or MCP.

This skill is focused on:

- brands
- campaigns
- posts
- schedules
- asset generation
- remix

Use it when you need to:

- create, list, update, or delete brands
- create, list, update, or delete campaigns
- create, list, update, or delete posts
- start asset generation jobs and poll them
- remix existing posts
- list linked social accounts
- schedule, reschedule, or cancel posts

Do not use this skill for:

- mailbox setup
- email triage
- replying to customer emails
- sending new outbound emails

For those, use `EMAIL_SKILL.md`.

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

## Authentication

All requests use:

```http
Authorization: Bearer <API_KEY>
```

## REST quick reference

Base URL:

```text
https://<your-convex-site>/api/v1
```

Core routes:

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
- `GET /openapi.json`

## MCP quick reference

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

1. List or create the brand.
2. Create or find the campaign.
3. Create or find the post.
4. If needed, list social accounts before scheduling.
5. Schedule the post.
6. If generating assets, start the job and poll until complete.

## Example workflow

Goal: create a brand from a website, create a campaign, generate content, and schedule the result.

1. `research_brand_from_website`
2. `create_campaign`
3. `start_asset_generation`
4. `get_asset_generation_job`
5. `list_posts`
6. `list_social_accounts`
7. `schedule_post`

## Best practices

- Prefer MCP when an agent needs to reason over structured tool output.
- Prefer REST for scripts, CI, or backend integrations.
- Reuse returned IDs exactly as given.
- Read `market-kit://openapi` or `/api/v1/openapi.json` before writing new integrations.
- Use workflow examples instead of inferring multi-step content flows from scratch.
