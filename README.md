# Market Kit Developer Docs

Public documentation for using the Market Kit developer surface.

This repo contains:

- `SKILL.md` as the top-level skill router for AI agents using Market Kit
- `POSTS_SKILL.md` for post, campaign, scheduling, and asset-generation workflows
- `EMAIL_SKILL.md` for mailbox, inbox, search, draft, reply, and send-email workflows
- `INSTALL.md` for setting up the REST API and the remote Streamable HTTP MCP server

## What Market Kit exposes

- A REST API under `/api/v1`
- A remote Streamable HTTP MCP server under `/mcp`
- First-class Mail support for mailbox management, inbox-style email listing, search, drafts, AI replies, replying, and sending new emails
- OpenAPI examples plus workflow examples for common REST and MCP sequences

Both use the same bearer API key.

## Files

- `SKILL.md`
- `POSTS_SKILL.md`
- `EMAIL_SKILL.md`
- `INSTALL.md`

## Notes

- All example values in this repo are placeholders.
- No secrets, deploy keys, or private environment values are included here.
