---
name: lark-email
description: Lark Email operations — send, read, and manage emails, folders, and mail contacts. Use when working with Lark Mail / email functionality.
metadata:
  version: "1.0.0"
  author: frnd-team
  tags: ["lark", "email", "mail", "mailbox", "contacts"]
---

# Lark Email Skill

Use this skill when working with Lark Mail — sending emails, reading inbox, managing folders, and maintaining mail contacts.

## Available Tools

| Tool | Purpose | Key Args |
|------|---------|----------|
| `send_email` | Send an email from user's mailbox | `subject`, `to`, `cc`, `bcc`, `body_html`, `body_plain_text` |
| `list_emails` | List email IDs in a folder | `folder_id`, `page_size`, `only_unread` |
| `get_email` | Get full email details | `message_id` |
| `get_email_by_card` | Get emails linked to an IM email card | `card_id`, `owner_id` |
| `get_email_attachment_url` | Get download URLs for attachments | `message_id`, `attachment_ids` |
| `create_email_folder` | Create a new mailbox folder | `name`, `parent_folder_id` |
| `delete_email_folder` | Delete a mailbox folder | `folder_id` |
| `list_email_folders` | List all mailbox folders | `folder_type` |
| `patch_email_folder` | Rename or move a folder | `folder_id`, `name`, `parent_folder_id` |
| `create_mail_contact` | Add a mail contact | `name`, `mail_address`, `company`, `phone` |
| `delete_mail_contact` | Remove a mail contact | `mail_contact_id` |
| `list_mail_contacts` | List mail contacts | `page_size`, `page_token` |
| `patch_mail_contact` | Update a mail contact | `mail_contact_id`, `name`, `mail_address` |
| `query_mail_user` | Look up Lark user IDs by email/phone | `emails`, `mobiles`, `user_id_type` |

## Workflows

### Send an Email

1. Use `send_email(subject="...", to="recipient@example.com", body_plain_text="...")` to send
2. Optionally include `cc`, `bcc`, and `body_html` for richer emails
3. Returns `message_id` and `thread_id` on success

### Read Inbox Emails

1. Use `list_emails(folder_id="INBOX")` to get email IDs from the inbox
2. Use `get_email(message_id=...)` to retrieve full details for each email
3. Body content (`body_html`, `body_plain_text`) is base64url-encoded in the response
4. For unread-only: `list_emails(folder_id="INBOX", only_unread=True)`

### Download Attachments

1. Use `get_email(message_id=...)` to get the attachment list with IDs
2. Use `get_email_attachment_url(message_id=..., attachment_ids="id1,id2")` to get download URLs
3. Download links are valid for 2 hours and can only be used twice

### Process Email Cards from IM

1. When an email card message is received in chat, extract `card_id` and `owner_id`
2. Use `get_email_by_card(card_id=..., owner_id=...)` to get associated email IDs
3. Use `get_email(message_id=...)` to read each email's content

### Manage Folders

1. Use `list_email_folders()` to see all folders (system + custom)
2. Use `create_email_folder(name="My Folder", parent_folder_id="0")` to create at root level
3. Use `patch_email_folder(folder_id=..., name="New Name")` to rename
4. Use `delete_email_folder(folder_id=...)` to delete (emails move to Trash)

### Manage Mail Contacts

1. Use `list_mail_contacts()` to browse the user's mail address book
2. Use `create_mail_contact(name="Alice", mail_address="alice@example.com")` to add
3. Use `patch_mail_contact(mail_contact_id=..., phone="...")` to update fields
4. Use `delete_mail_contact(mail_contact_id=...)` to remove

### Resolve Email to Lark User

1. Use `query_mail_user(emails="user@example.com")` to find the Lark user ID
2. Returns `open_id`, `user_id`, or `union_id` based on `user_id_type`
3. Does not support enterprise email addresses — use personal email

## Important Notes

- **user_mailbox_id**: Most tools accept `user_mailbox_id` which defaults to `"me"` (current authenticated user). You can also pass a specific email address like `"user@company.com"`.
- **All tools require user_access_token** — they operate on behalf of the authenticated user, not the bot.
- **list_emails returns only IDs** — you must call `get_email` separately to get email content.
- **folder_id for inbox**: Use the string `"INBOX"` as the folder_id to list inbox messages.
- **base64url encoding**: Email body fields (`body_html`, `body_plain_text`) in get_email responses are base64url-encoded. Decode before displaying.
- **Attachment download URLs** are single-use and expire after 2 hours.
- **query_mail_user** uses the Contacts API (not Mail API) and does not support enterprise email addresses.
- **Rate limits** vary by endpoint: send_email is 100/min, list_emails is 10/s, get_email is 100/min, folder ops are 1-5/s, contact ops are 20/min.
