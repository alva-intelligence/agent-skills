---
name: lark-bitable
description: Lark Bitable (Base) operations -- manage multidimensional tables, records, fields, views, and dashboards. Use for any structured data, project tracking, CRM, or database-like operations in Lark.
metadata:
  version: "1.0.0"
  author: frnd-team
  tags: ["lark", "bitable", "base", "database", "records", "tables"]
---

# Lark Bitable Skill

Use this skill when working with Lark Bitable (Base) -- the multidimensional spreadsheet/database product.

## Available Tools

### App Tools

| Tool | Purpose | Key Args |
|------|---------|----------|
| `get_app` | Get Bitable app metadata (name, revision, permissions) | `app_token` |
| `create_app` | Create a new Bitable app in a folder | `name`, `folder_token` |
| `update_app` | Update app metadata (name, advanced permissions) | `app_token`, `name`, `is_advanced` |

### Table Tools

| Tool | Purpose | Key Args |
|------|---------|----------|
| `list_tables` | List all data tables in an app | `app_token`, `page_size`, `page_token` |
| `create_table` | Create a new data table with optional fields | `app_token`, `name`, `fields_json` |
| `delete_table` | Delete a data table | `app_token`, `table_id` |
| `patch_table` | Rename a data table | `app_token`, `table_id`, `name` |

### Field Tools

| Tool | Purpose | Key Args |
|------|---------|----------|
| `list_fields` | List all fields (columns) in a table | `app_token`, `table_id` |
| `create_field` | Add a new field to a table | `app_token`, `table_id`, `field_name`, `type` |
| `update_field` | Update a field definition (full replacement) | `app_token`, `table_id`, `field_id`, `field_name`, `type` |
| `delete_field` | Delete a field from a table | `app_token`, `table_id`, `field_id` |

### View Tools

| Tool | Purpose | Key Args |
|------|---------|----------|
| `list_views` | List all views in a table | `app_token`, `table_id` |
| `get_view` | Get a specific view by ID | `app_token`, `table_id`, `view_id` |
| `create_view` | Create a new view (grid, kanban, gallery, gantt, form) | `app_token`, `table_id`, `view_name`, `view_type` |
| `delete_view` | Delete a view | `app_token`, `table_id`, `view_id` |
| `patch_view` | Update a view's name or properties | `app_token`, `table_id`, `view_id`, `view_name` |

### Record Tools

| Tool | Purpose | Key Args |
|------|---------|----------|
| `get_record` | Get a single record by ID | `app_token`, `table_id`, `record_id` |
| `list_records` | List records with pagination | `app_token`, `table_id`, `page_size`, `page_token` |
| `search_records` | Query records with filter and sort conditions | `app_token`, `table_id`, `filter_json`, `sort_json` |
| `create_record` | Create a single record | `app_token`, `table_id`, `fields_json` |
| `update_record` | Update a single record (incremental) | `app_token`, `table_id`, `record_id`, `fields_json` |
| `delete_record` | Delete a single record | `app_token`, `table_id`, `record_id` |
| `batch_create_records` | Create up to 500 records at once | `app_token`, `table_id`, `records_json` |
| `batch_update_records` | Update up to 500 records at once | `app_token`, `table_id`, `records_json` |

### Dashboard Tools

| Tool | Purpose | Key Args |
|------|---------|----------|
| `list_dashboards` | List all dashboards in an app | `app_token`, `page_size`, `page_token` |

## Workflows

### Explore a Bitable App

1. Use `get_app(app_token)` to get app metadata
2. Use `list_tables(app_token)` to see all tables
3. Use `list_fields(app_token, table_id)` to see columns/fields for a table
4. Use `list_views(app_token, table_id)` to see available views

### Find and Read Records

1. Use `list_tables(app_token)` to find the correct table
2. Use `list_fields(app_token, table_id)` to understand the schema
3. Use `search_records(app_token, table_id, filter_json=...)` to find specific records
4. Use `get_record(app_token, table_id, record_id)` for a single record's full details

### CRUD Records

1. **Create**: `create_record(app_token, table_id, fields_json='{"Name":"Alice","Status":"Active"}')`
2. **Read**: `search_records(app_token, table_id, filter_json='{"conjunction":"and","conditions":[{"field_name":"Name","operator":"is","value":["Alice"]}]}')`
3. **Update**: `update_record(app_token, table_id, record_id, fields_json='{"Status":"Done"}')`
4. **Delete**: `delete_record(app_token, table_id, record_id)`

### Batch Operations

1. Prepare a JSON array of record objects
2. Use `batch_create_records(app_token, table_id, records_json='[{"fields":{"Name":"A"}},{"fields":{"Name":"B"}}]')`
3. Use `batch_update_records(app_token, table_id, records_json='[{"record_id":"recXXX","fields":{"Status":"Done"}}]')`

### Set Up a New Table with Schema

1. Use `create_table(app_token, name, fields_json='[{"field_name":"Name","type":1},{"field_name":"Age","type":2}]')`
2. Add additional fields with `create_field(app_token, table_id, field_name, type)`
3. Create views as needed with `create_view(app_token, table_id, view_name, view_type)`

## Field Types Reference

| Type | Name | Value Format |
|------|------|-------------|
| 1 | Text | String |
| 2 | Number | Number |
| 3 | SingleSelect | String (option value) |
| 4 | MultiSelect | Array of strings |
| 5 | DateTime | Millisecond timestamp (e.g. 1674206443000) |
| 7 | Checkbox | Boolean (true/false) |
| 11 | Person | Array of `{"id":"ou_xxx"}` |
| 13 | Phone | String |
| 15 | URL/Hyperlink | `{"text":"label","link":"https://..."}` |
| 17 | Attachment | Array of `{"file_token":"xxx"}` |
| 18 | SingleLink | Array of record IDs |
| 21 | DuplexLink | Array of record IDs |
| 22 | Location | `"longitude,latitude"` string |
| 23 | GroupChat | Array of `{"id":"oc_xxx"}` |
| 1001 | CreatedTime | Auto (read-only) |
| 1002 | ModifiedTime | Auto (read-only) |
| 1003 | CreatedBy | Auto (read-only) |
| 1004 | ModifiedBy | Auto (read-only) |
| 1005 | AutoNumber | Auto (read-only) |

## Search Filter Syntax

The `filter_json` parameter for `search_records` uses this structure:

```json
{
  "conjunction": "and",
  "conditions": [
    {
      "field_name": "Status",
      "operator": "is",
      "value": ["Active"]
    },
    {
      "field_name": "Amount",
      "operator": "isGreater",
      "value": ["1000"]
    }
  ]
}
```

**Operators**: `is`, `isNot`, `contains`, `doesNotContain`, `isEmpty`, `isNotEmpty`, `isGreater`, `isGreaterEqual`, `isLess`, `isLessEqual`

**Conjunction**: `and` (all conditions) or `or` (any condition)

## Important Notes

- **Batch limits**: max 500 records per `batch_create_records` call, max 500 per `batch_update_records` call
- **Pagination**: use `page_token` from responses to fetch next pages; `search_records` returns max 500 per page
- **Update is incremental**: `update_record` only modifies provided fields; set a field to `null` to clear it
- **Update field is full replace**: `update_field` completely replaces the field definition including properties
- **No concurrent writes**: Bitable does not support concurrent write operations on the same table; serialize write requests
- **Date fields**: always use millisecond timestamps (not seconds)
- **Person fields**: use `[{"id":"ou_xxx"}]` format with open_id, user_id, or union_id
- **app_token from URL**: for base URLs (`feishu.cn/base/XXX`), the token is in the URL; for wiki URLs (`feishu.cn/wiki/XXX`), use the wiki API to resolve the token
- **Resource limits per app**: max 100 tables+dashboards, max 200 views per table, max 300 fields per table

## Search Best Practices

When using `search_records`:

- **TOTAL BUDGET**: Maximum 8 search/lookup API calls PER TASK
- **PER-QUERY**: Max 3 search calls for a single piece of information
- Each search MUST use a DIFFERENT strategy (different filter conditions, not slight variations):
  - ❌ 'Status=Active' → 'Status="Active "' → 'status=active' (too similar!)
  - ✅ 'Status=Active' → 'Assignee=John' (different field) → 'Created>2024-01-01' (different angle)

**WHEN SEARCH FAILS (2-3 attempts with no results):**
- STOP searching and return results to main agent
- Include: what you filtered for, what conditions you tried
- Let the main agent decide next steps

**WHEN YOU ALREADY FOUND IT:**
- If search returned records, do NOT search again with different filters
- Use what you found
