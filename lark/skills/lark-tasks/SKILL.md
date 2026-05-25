---
name: lark-tasks
description: Lark Task v2 operations — task CRUD, members, reminders, comments, and attachments. Use when working with Lark tasks, to-dos, or project management.
metadata:
  version: "1.0.0"
  author: frnd-team
  tags: ["lark", "tasks", "todo", "project-management", "comments", "attachments"]
---

# Lark Tasks Skill

Use this skill when working with Lark Task v2 — creating, managing, and tracking tasks, adding comments, and viewing attachments.

## Available Tools

| Tool | Purpose | Key Args |
|------|---------|----------|
| `list_tasks` | List tasks assigned to the caller ("my tasks") | `page_size`, `page_token`, `completed` |
| `get_task` | Get full task details by GUID | `task_guid` |
| `create_task` | Create a new task | `summary`, `description`, `due_timestamp`, `members` |
| `patch_task` | Update task fields (partial update) | `task_guid`, `update_fields`, `summary`, `due_timestamp`, etc. |
| `delete_task` | Permanently delete a task | `task_guid` |
| `add_task_members` | Add assignees or followers | `task_guid`, `members` |
| `add_task_reminders` | Set a reminder relative to due time | `task_guid`, `relative_fire_minute` |
| `list_comments` | List comments on a task | `resource_id`, `direction`, `page_size` |
| `create_comment` | Add a comment to a task | `resource_id`, `content` |
| `delete_comment` | Delete a comment | `comment_id` |
| `list_attachments` | List attachments on a task | `resource_id`, `page_size` |

## Workflows

### Create a Task with Assignees and Reminders

1. Use `create_task(summary="...", due_timestamp="1684652400000", members=[{"id": "ou_xxx", "role": "assignee"}])` to create the task with an initial assignee
2. Use `add_task_members(task_guid, members=[{"id": "ou_yyy", "role": "follower"}])` to add followers
3. Use `add_task_reminders(task_guid, relative_fire_minute=30)` to remind 30 min before due

### Update a Task

1. Use `patch_task(task_guid, update_fields=["summary", "due"], summary="New title", due_timestamp="1684652400000")` to update specific fields
2. Only fields listed in `update_fields` are modified; omitted fields are unchanged
3. To mark a task complete: `patch_task(task_guid, update_fields=["completed_at"], completed_at="1684652400000")`
4. To reopen a task: `patch_task(task_guid, update_fields=["completed_at"], completed_at="0")`

### View Task and Comments

1. Use `get_task(task_guid)` to get full task details
2. Use `list_comments(resource_id=task_guid, direction="desc")` to get latest comments
3. Use `create_comment(resource_id=task_guid, content="Status update...")` to add a comment

### Browse My Tasks

1. Use `list_tasks()` to get all tasks you are responsible for
2. Filter with `completed="false"` for only open tasks, or `completed="true"` for done tasks
3. Use `page_token` from the response to paginate through results

## Important Notes

- **Timestamps are in milliseconds** (not seconds). All time fields use string-encoded millisecond timestamps since epoch (e.g. `"1684652400000"`).
- **Due/start date format**: Use `{"timestamp": "...", "is_all_day": true}` for full-day dates, `false` for specific times. If both start and due are set, `is_all_day` must match.
- **update_fields pattern**: When using `patch_task`, you MUST include every field you want to change in the `update_fields` list. A field in `update_fields` but not in the task body will be cleared.
- **Member types**: `"user"` (default) or `"app"`. Roles for tasks: `"assignee"` (can edit/complete) or `"follower"` (receives notifications).
- **Reminders**: Each task supports at most 1 reminder. To change an existing reminder, remove it first then add a new one. The task must have a due date set.
- **Task identifiers**: Tasks use a GUID (e.g. `"d300a75f-c56a-4be9-80d1-e47653028ceb"`), not the `task_id` shown in the UI (e.g. `"t1002325"`).
- **Comments**: Use `resource_id` (task GUID) and `resource_type="task"` for all comment operations.
- **Attachments**: `list_attachments` returns temporary download URLs valid for 3 minutes (max 3 downloads each).
- **Pagination**: Max `page_size` is 100, default is 50. Check `has_more` in the response before requesting the next page.
- **Permissions**: `tenant_access_token` has no special privileges -- it can only access tasks it created or was added to as a member.

## Search Best Practices

When using `list_tasks`:

- **TOTAL BUDGET**: Maximum 8 search/lookup API calls PER TASK
- **PER-QUERY**: Max 3 search calls for a single piece of information
- Each search MUST use a DIFFERENT filter strategy:
  - ❌ 'my tasks' → 'my tasks' → 'my tasks' (repetitive!)
  - ✅ 'list_tasks completed=false' → 'list_tasks assignee=John' → 'list_tasks due>2024-01-01' (different angle)

**WHEN SEARCH FAILS (2-3 attempts with no results):**
- STOP searching and return results to main agent
- Include: what filters you tried, what task you were looking for
- Let the main agent decide next steps

**WHEN YOU ALREADY FOUND IT:**
- If tasks were found, do NOT search again with different filters
- Use what you found
