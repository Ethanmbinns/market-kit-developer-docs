# Market Kit Developer API and MCP Skill

Use this file as the top-level router for Market Kit skills.

Market Kit now exposes two primary agent workflows, and they work better when they are documented and loaded separately:

- `POSTS_SKILL.md`
- `EMAIL_SKILL.md`

## Recommended split

Use `POSTS_SKILL.md` when the agent is focused on:

- brands
- campaigns
- posts
- schedules
- asset generation
- remix

Use `EMAIL_SKILL.md` when the agent is focused on:

- mailboxes
- folders
- inbox listing
- search
- drafts
- replies
- sending new emails

This keeps agent context smaller and makes the operational model much clearer. It also reduces the chance that an automation meant for content creation starts reasoning from mail-specific tools, or vice versa.

## Shared contract

Both skills use the same developer surface:

- REST API under `/api/v1`
- Streamable HTTP MCP server under `/mcp`
- bearer API keys
- stable public IDs

If an agent needs both domains in one run, load both skills. Otherwise, prefer the narrower skill that matches the task.

## Start here

Before writing automation against Market Kit:

1. Read `INSTALL.md`
2. Choose `POSTS_SKILL.md`, `EMAIL_SKILL.md`, or both
3. Read `market-kit://openapi` or fetch `/api/v1/openapi.json`
4. Read `market-kit://docs/examples`

## Non-goals

These skills do not cover:

- auth sign-in flows
- billing or admin features
- social account OAuth linking setup

Those remain outside the public developer surface.
