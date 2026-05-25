# lark/

Lark-platform agent skills — Bitable, Calendar, Card formatting, Docs/Wiki, Drive, Email, IM/Contacts, Sheets, Tasks.

These skills wrap the Lark Open Platform APIs into reusable agent instructions. Originally lived in `alva-intelligence/frnd-lark` at `api/skills/`; migrated here so any agent (not just the FRnD bot) can install + use them.

## Skills

| Skill | Purpose |
|-------|---------|
| [`lark-bitable`](./skills/lark-bitable) | Bitable (multi-dim table) CRUD: tables, fields, records, views. |
| [`lark-calendar`](./skills/lark-calendar) | Calendar + events: create/update/list, free-busy, room booking. |
| [`lark-card-formatting`](./skills/lark-card-formatting) | Lark Card JSON 2.0 markdown rules — rich text, collapsible panels. Load BEFORE writing bot responses. |
| [`lark-docs-wiki`](./skills/lark-docs-wiki) | Docs + Wiki: create, read, edit; manage wiki nodes + permissions. |
| [`lark-drive`](./skills/lark-drive) | Drive: upload/download, file metadata, folder ops, permissions. |
| [`lark-email`](./skills/lark-email) | Email: send/reply/forward, drafts, folders, labels, attachments. |
| [`lark-im-contacts`](./skills/lark-im-contacts) | IM + contacts: send messages, manage chats, query users/departments. |
| [`lark-sheets`](./skills/lark-sheets) | Sheets: read/write cells, append rows, formulas, sheet management. |
| [`lark-tasks`](./skills/lark-tasks) | Tasks: create/assign/update todos, subtasks, lists. |

## Agents

_None yet._

## Source

Skills migrated from `alva-intelligence/frnd-lark` (commit history preserved in original repo). Future updates land here.
