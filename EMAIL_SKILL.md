# Market Kit Email Skill

Use this skill when you need to operate Market Kit's mailbox and inbox side through REST or MCP.

This skill is focused on:

- mailboxes
- folders
- inbox-style email listing
- search
- thread detail
- drafts
- AI reply drafting
- replies
- brand-new outbound email

Use it when you need to:

- create, test, update, or delete mailboxes
- list mailbox folders
- list emails with stable folder and view filters
- search mailbox emails
- open an email thread
- save or clear a draft
- generate an AI reply draft
- reply to an existing thread
- send a brand-new email

Do not use this skill for:

- brand creation
- campaign creation
- post scheduling
- asset generation
- remix

For those, use `POSTS_SKILL.md`.

## Mental model

The higher-level `/emails` layer is the primary agent-facing mail surface.

Prefer:

- `/mailboxes/{mailboxId}/emails`
- `/mailboxes/{mailboxId}/emails/search`
- `/mailboxes/{mailboxId}/emails/{threadId}`
- `/mailboxes/{mailboxId}/emails/{threadId}/reply`
- `/mailboxes/{mailboxId}/emails/send`

The older `/threads` routes still exist for compatibility and draft/AI reply plumbing, but agents should start with the `/emails` layer.

Supported system folders:

- `all`
- `inbox`
- `sent`
- `archive`
- `spam`
- `drafts`

Supported views:

- `all`
- `unread`
- `new`
- `waiting_for_reply`
- `needs_reply`

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
- `GET /mailboxes/{mailboxId}/threads/{threadId}/draft`
- `PUT /mailboxes/{mailboxId}/threads/{threadId}/draft`
- `DELETE /mailboxes/{mailboxId}/threads/{threadId}/draft`
- `POST /mailboxes/{mailboxId}/threads/{threadId}/ai-reply`
- `GET /openapi.json`

## MCP quick reference

MCP resources:

- `market-kit://openapi`
- `market-kit://docs/examples`

MCP tools:

- `list_mailboxes`
- `discover_mailbox_settings`
- `test_mailbox_settings`
- `create_mailbox`
- `get_mailbox`
- `update_mailbox`
- `delete_mailbox`
- `create_mail_attachment_upload_url`
- `list_mail_folders`
- `list_emails`
- `search_emails`
- `get_email_thread`
- `get_mail_draft`
- `save_mail_draft`
- `clear_mail_draft`
- `generate_ai_mail_reply`
- `reply_to_email`
- `send_email`

## Recommended workflow

1. Find or create the mailbox.
2. List emails with `folder` and `view`.
3. Use `search_emails` when you need sender/subject/free-text narrowing.
4. Open the thread with `get_email_thread`.
5. If needed, generate an AI reply and save a draft.
6. Reply to the thread or send a brand-new email.

## Example workflow

Goal: triage the inbox, find an order-related email, generate a reply, and send it.

1. `list_emails`
2. `search_emails`
3. `get_email_thread`
4. `generate_ai_mail_reply`
5. `save_mail_draft`
6. `reply_to_email`

Goal: send a brand-new outbound email.

1. `send_email`

## Best practices

- Prefer `folder` over raw `folderPath` unless you specifically need a provider IMAP path.
- Treat `/emails` as the default inbox surface.
- Use `view=all&folder=all` for the broadest mailbox pull.
- Use `market-kit://openapi` or `/api/v1/openapi.json` when payload shape is unclear.
- Keep IDs exact and never invent `mailbox_...` or `thread_...` values.
