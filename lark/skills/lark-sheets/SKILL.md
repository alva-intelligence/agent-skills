---
name: lark-sheets
description: Lark Sheets operations — spreadsheet management, worksheet queries, cell find/replace, and row/column movement. Use when working with Lark spreadsheets.
metadata:
  version: "1.0.0"
  author: frnd-team
  tags: ["lark", "sheets", "spreadsheet", "worksheet", "data"]
---

# Lark Sheets Skill

Use this skill when working with Lark spreadsheets (creating, reading, searching, and organizing sheet data).

## Available Tools

| Tool | Purpose | Key Args |
|------|---------|----------|
| `get_spreadsheet` | Get spreadsheet info (title, URL, metadata) | `spreadsheet_token` |
| `create_spreadsheet` | Create a new spreadsheet in a folder | `title`, `folder_token` |
| `patch_spreadsheet` | Update spreadsheet title | `spreadsheet_token`, `title` |
| `get_sheet` | Get a single worksheet's properties | `spreadsheet_token`, `sheet_id` |
| `query_sheet` | List all worksheets in a spreadsheet | `spreadsheet_token` |
| `find_in_sheet` | Search for text/regex in a cell range | `spreadsheet_token`, `sheet_id`, `find`, `range` |
| `replace_in_sheet` | Find and replace text in a cell range | `spreadsheet_token`, `sheet_id`, `find`, `replacement`, `range` |
| `move_dimension` | Move rows or columns to a new position | `spreadsheet_token`, `sheet_id`, `major_dimension`, `start_index`, `end_index`, `destination_index` |

## Workflows

### Explore a Spreadsheet

1. Use `get_spreadsheet(spreadsheet_token)` to get title and metadata
2. Use `query_sheet(spreadsheet_token)` to list all worksheets with their IDs, titles, row/column counts
3. Use `get_sheet(spreadsheet_token, sheet_id)` for detailed info on a specific worksheet

### Create and Set Up a Spreadsheet

1. Use `create_spreadsheet(title, folder_token)` to create a new spreadsheet
2. The response includes the new `spreadsheet_token` and URL
3. Use `query_sheet(spreadsheet_token)` to get the default worksheet's `sheet_id`

### Search and Replace Data

1. Use `find_in_sheet(spreadsheet_token, sheet_id, find, range)` to locate cells
2. Optionally set `match_case=True` for case-sensitive search or `search_by_regex=True` for regex
3. Use `replace_in_sheet(...)` with the same params plus `replacement` to do find-and-replace

### Reorganize Rows or Columns

1. Use `move_dimension(spreadsheet_token, sheet_id, major_dimension="ROWS", start_index, end_index, destination_index)` to reorder rows
2. Use `major_dimension="COLUMNS"` to move columns instead
3. All indices are 0-based; moved content shifts existing rows/columns down or right

## Important Notes

- **Cell range syntax**: `<sheetId>!<start>:<end>`, e.g. `"abc123!A1:C5"`, `"abc123!A:B"`, or just `"abc123"` for entire sheet
- **spreadsheet_token** can be extracted from a Lark sheet URL: `https://sample.larksuite.com/sheets/<spreadsheet_token>`
- **sheet_id** can be found in the URL query param `?sheet=<sheet_id>` or from `query_sheet` / `get_sheet` responses
- `find_in_sheet` range must not exceed the actual data area, or it returns error 1310202
- `replace_in_sheet` is limited to 5,000 cells per call and 20 requests/minute
- `move_dimension` indices are 0-based (start_index=0 means first row/column)
- `create_spreadsheet` is rate-limited to 20 requests/minute
- For structured/tabular data with field types, filtering, and views, consider using **Bitable** (multi-dimensional tables) instead of Sheets

## Search Best Practices

When using `find_in_sheet` or `query_sheet`:

- **TOTAL BUDGET**: Maximum 8 search/lookup API calls PER TASK
- **PER-QUERY**: Max 3 search calls for a single piece of information
- Each search MUST use a DIFFERENT strategy:
  - ❌ 'find "sales"' → 'find "Sales"' → 'find "SALES"' (too similar!)
  - ✅ 'query_sheet for range A1:Z100' → 'find_in_sheet for "sales"' → 'get_sheet to see sheet names' (different angle)

**WHEN SEARCH FAILS (2-3 attempts with no results):**
- STOP searching and return results to main agent
- Include: what you searched for, what ranges you checked
- Let the main agent decide next steps

**WHEN YOU ALREADY FOUND IT:**
- If data was found, do NOT search again
- Use what you found
