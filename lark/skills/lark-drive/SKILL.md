---
name: lark-drive
description: Lark Drive operations — file/folder management and permission control. Use when working with Lark cloud documents, folders, sharing permissions, or download URLs.
metadata:
  version: "1.0.0"
  author: frnd-team
  tags: ["lark", "drive", "files", "folders", "permissions", "sharing"]
---

# Lark Drive Skill

Use this skill when working with Lark Drive file/folder management and permission operations.

## Available Tools

### File / Folder Operations

| Tool | Purpose | Key Args |
|------|---------|----------|
| `list_files` | List files in a folder | `folder_token`, `order_by`, `direction`, `page_size` |
| `create_folder` | Create a new folder | `name`, `folder_token` |
| `copy_file` | Copy a file | `file_token`, `name`, `type`, `folder_token` |
| `move_file` | Move a file or folder | `file_token`, `type`, `folder_token` |
| `delete_file` | Delete a file or folder | `file_token`, `type` |
| `download_drive_file` | Download & index file from Drive folder (PDF, etc.) | `file_token`, `index` |

### Permission — Members

| Tool | Purpose | Key Args |
|------|---------|----------|
| `create_permission` | Add a collaborator | `token`, `type`, `member_type`, `member_id`, `perm` |
| `batch_create_permissions` | Batch add collaborators (max 10) | `token`, `type`, `members` |
| `list_permissions` | List all collaborators | `token`, `type` |
| `update_permission` | Update a collaborator's role | `token`, `member_id`, `type`, `member_type`, `perm` |
| `delete_permission` | Remove a collaborator | `token`, `member_id`, `type`, `member_type` |
| `transfer_owner` | Transfer document ownership | `token`, `type`, `member_type`, `member_id` |

### Permission — Public Settings

| Tool | Purpose | Key Args |
|------|---------|----------|
| `get_public_permission` | Get link-sharing settings | `token`, `type` |
| `patch_public_permission` | Update link-sharing settings | `token`, `type`, `settings` |

### Media

| Tool | Purpose | Key Args |
|------|---------|----------|
| `batch_get_download_urls` | Get temporary download URLs | `file_tokens`, `extra` |

## Workflows

### Browse and Organize Files

1. Use `list_files(folder_token)` to list contents of a folder (empty = root)
2. Use `create_folder(name, folder_token)` to create subfolders
3. Use `move_file(file_token, type, folder_token)` to reorganize files
4. Use `copy_file(file_token, name, type, folder_token)` to duplicate files
5. Use `delete_file(file_token, type)` to remove unwanted items

### Permission Management

1. Use `list_permissions(token, type)` to see current collaborators
2. Use `create_permission(token, type, member_type, member_id, perm)` to add one
3. Use `batch_create_permissions(token, type, members)` to add multiple (max 10)
4. Use `update_permission(token, member_id, type, member_type, perm)` to change roles
5. Use `delete_permission(token, member_id, type, member_type)` to revoke access

### Configure Link Sharing

1. Use `get_public_permission(token, type)` to check current sharing settings
2. Use `patch_public_permission(token, type, settings)` to update settings, e.g.:
   - Enable org-wide read: `{"link_share_entity": "tenant_readable"}`
   - Disable external sharing: `{"external_access_entity": "closed"}`
   - Restrict downloads: `{"copy_entity": "only_full_access"}`

### Transfer Document Ownership

1. Use `transfer_owner(token, type, member_type="openid", member_id=new_owner_id)`
2. Optionally set `remove_old_owner=True` or `old_owner_perm="view"` to downgrade

### Get Download URLs

1. Use `batch_get_download_urls(file_tokens=["token1", "token2"])` to get temp URLs
2. For bitable attachments with advanced permissions, pass the `extra` JSON string

## Important Notes

- **File types**: `doc`, `docx`, `sheet`, `bitable`, `file`, `folder`, `mindnote`, `minutes`, `slides`
- **Token vs file_token**: Permission APIs use `token` (generic doc token); file APIs use `file_token`
- **`type` is required**: Most permission endpoints and `delete_file` require the `type` query param matching the document type
- **Permission roles**: `view` (read-only), `edit` (can edit), `full_access` (can manage)
- **Member types**: `openid`, `email`, `userid`, `unionid`, `openchat` (group), `opendepartmentid`, `groupid`
- **Pagination**: `list_files` supports max 200 per page; use `page_token` for next page
- **Public permission API is v2**: ` and `patch_publicget_public_permission`_permission` use the v2 API path
- **Batch limits**: `batch_create_permissions` accepts max 10 members per call
- **Folder tokens** can be obtained from `list_files` response or from the browser URL

## External File Processing Workflow

When ANY agent finds external files (type="file" from search results, Drive listings, or any other source):

### Step 1: Try Download First
1. Use `batch_get_download_urls(file_tokens='<token>')` to get temporary download URL
2. If successful → Process the file content for the user's query
3. Return the summarized/processed result to main agent

### Step 2: If Download Fails → Report Back
If download fails (permission denied, token expired, or other errors):
1. Report to main agent: "Download failed for file [name]. User may need to re-upload as Lark Doc OR grant Drive access."
2. The main agent can then decide: ask user to re-upload, or try alternative approaches

### Step 3: For Message Attachments (Alternative)
If the file was shared via chat message (not Drive):
- The main agent can use `download_message_attachment(message_id, file_key, download=True, index=True)`
- This will download AND index the file into RAG knowledge base
- Then re-query the knowledge base for the answer

## Search Best Practices

When using `list_files`:

- **TOTAL BUDGET**: Maximum 8 search/lookup API calls PER TASK
- **PER-QUERY**: Max 3 calls for a single piece of information
- Each search MUST use a DIFFERENT strategy:
  - ❌ 'list_files in folder A' → 'list_files in folder A' → 'list_files in folder A' (repetitive!)
  - ✅ 'list_files in folder A' → 'list_files in subfolder B' → 'search by name in folder A' (different angle)

**WHEN SEARCH FAILS (2-3 attempts with no results):**
- STOP searching and return results to main agent
- Include: what folder you checked, what file you were looking for
- Let the main agent decide next steps

**WHEN YOU ALREADY FOUND IT:**
- If files were found, do NOT search again
- Use what you found
