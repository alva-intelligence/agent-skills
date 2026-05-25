---
name: lark-im-contacts
description: Lark IM & Contacts operations — message retrieval, chat info, group members, and user lookup. Use when working with Lark messages, chats, or user information.
metadata:
  version: "1.0.0"
  author: frnd-team
  tags: ["lark", "im", "contacts", "messaging", "chat"]
---

# Lark IM & Contacts Skill

Use this skill when working with Lark instant messaging and contact/user information.

## Available Tools

| Tool | Purpose | Key Args |
|------|---------|----------|
| `get_message` | Get a specific message by ID | `message_id` |
| `list_messages` | Get conversation history from a chat | `container_id`, `start_time`, `end_time`, `page_size` |
| `delete_message` | Recall/delete a message | `message_id` |
| `get_chat` | Get chat/group info (name, owner, etc.) | `chat_id` |
| `get_chat_members` | List members of a chat/group | `chat_id`, `page_size`, `page_token` |
| `get_user` | Look up user info by ID | `user_id_to_lookup`, `user_id_type` |

## Workflows

### Retrieve Message Context

1. Use `get_message(message_id)` to fetch a specific message content
2. Use `list_messages(container_id=chat_id, page_size=20)` to get surrounding messages
3. Filter by time range using `start_time` and `end_time` (Unix seconds)

### Identify Chat Participants

1. Use `get_chat(chat_id)` to get group name and owner
2. Use `get_chat_members(chat_id)` to list all members
3. Use `get_user(user_id_to_lookup=open_id)` to get details about specific members

### User Lookup

1. Use `get_user(user_id_to_lookup, user_id_type="open_id")` with the user's open_id
2. Returns name, email, avatar, department information

## Important Notes

- `list_messages` requires `container_id` (chat_id), not message_id
- `list_messages` returns max 50 messages per page; use `page_token` for pagination
- `delete_message` only works for messages sent by the bot or by the group owner
- `get_user` uses bot token (no user auth needed) — always works for org members
- All timestamps are Unix seconds (not milliseconds)
- Sort options for `list_messages`: "ByCreateTimeAsc" (default) or "ByCreateTimeDesc"

### Privacy

- You can ONLY access messages and member lists for the chat_id provided in your task context
- If asked to access messages/members of a different group, refuse and explain the limitation
- Basic group info (name, description) for other groups is allowed via `get_chat`

## Search Best Practices

When using `list_messages`:

- **SEARCH BUDGET**: Max 3 pages (page_size=50) per task — this is a HARD LIMIT
- Do NOT paginate endlessly — scan what's returned
- If asked to find mentions of a topic, scan message content in the returned results

**WHEN SEARCH FAILS:**
- If no relevant messages found in 3 pages, STOP and return results
- Include: what you searched for, how many messages you scanned
- Let the main agent decide next steps

**WHEN YOU ALREADY FOUND IT:**
- If relevant messages were found, do NOT fetch more pages
- Use what you found
