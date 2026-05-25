# Table Component Reference

The table component (`tag: "table"`) displays structured data with columns and rows. It supports text, markdown, options/tags, numbers, persons, and dates.

## JSON Structure

```json
{
  "tag": "table",
  "page_size": 5,
  "row_height": "low",
  "freeze_first_column": false,
  "header_style": {
    "text_align": "left",
    "text_size": "normal",
    "background_style": "none",
    "text_color": "grey",
    "bold": true,
    "lines": 1
  },
  "columns": [
    {
      "name": "col_key",
      "display_name": "Column Name",
      "width": "auto",
      "data_type": "text",
      "vertical_align": "center",
      "horizontal_align": "left"
    }
  ],
  "rows": [
    {
      "col_key": "value"
    }
  ]
}
```

## Key Fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `page_size` | Number | `5` | Rows per page, range [1, 10] |
| `row_height` | String | `low` | `low`, `middle`, `high`, or `[32,124]px` |
| `freeze_first_column` | Boolean | `false` | Freeze first column on horizontal scroll |
| `header_style.text_align` | String | `left` | `left`, `center`, `right` |
| `header_style.text_size` | String | `normal` | `normal` (14px), `heading` (16px) |
| `header_style.background_style` | String | `none` | `grey` or `none` |
| `header_style.text_color` | String | `default` | `default` or `grey` |
| `header_style.bold` | Boolean | `true` | Bold header text |
| `header_style.lines` | Number | `1` | Number of text lines in header |

## Column Definition

| Field | Required | Default | Description |
|-------|----------|---------|-------------|
| `name` | Yes | — | Unique key matching row data fields |
| `display_name` | No | empty | Header display text (empty = no header) |
| `width` | No | `auto` | `auto`, `[80,600]px`, or `[1,100]%` |
| `data_type` | Yes | `text` | Column data type (see below) |
| `vertical_align` | No | `center` | `top`, `center`, `bottom` |
| `horizontal_align` | No | `left` | `left`, `center`, `right` |

## Data Types

### `text` — Plain text (default)
```json
{ "name": "col1", "data_type": "text" }
// Row: "col1": "Hello world"
```

### `lark_md` — Partial markdown (links, bold)
```json
{ "name": "col2", "data_type": "lark_md" }
// Row: "col2": "[Link text](https://example.com)"
```

### `options` — Colored tag labels
```json
{ "name": "status", "data_type": "options" }
// Row single: "status": "Active"
// Row multi:
// "status": [
//   { "text": "S1", "color": "red" },
//   { "text": "S2", "color": "blue" }
// ]
```
Colors: `blue` (default), `red`, `green`, `orange`, `purple`, `yellow`, `turquoise`, `lime`, `violet`, `indigo`, `wathet`, `carmine`, `neutral`

### `number` — Numeric with optional formatting
```json
{
  "name": "amount",
  "data_type": "number",
  "format": {
    "symbol": "¥",
    "precision": 2,
    "separator": true
  }
}
// Row: "amount": 1234.56
```
- `symbol`: 1-char currency prefix (optional)
- `precision`: decimal places 0-10 (optional)
- `separator`: thousand separator commas (default `false`)

### `persons` — User avatars + names
```json
{ "name": "owner", "data_type": "persons" }
// Row single: "owner": "ou_xxxx"
// Row multi: "owner": ["ou_xxxx", "ou_yyyy"]
```
Accepts `open_id`, `user_id`, `union_id`.

### `date` — Date/time from Unix timestamp
```json
{
  "name": "due_date",
  "data_type": "date",
  "date_format": "YYYY/MM/DD"
}
// Row: "due_date": 1699341315000  (milliseconds)
```
Format placeholders: `YYYY`, `MM`, `DD`, `HH`, `mm`, `ss`

### `markdown` — Full markdown (images, code blocks)
```json
{ "name": "preview", "data_type": "markdown" }
// Row: "preview": "![image](img_key)"
```

## Limits & Rules

- Max **50 columns** per table
- Max **5 tables** per card
- Max **10 rows** per page (`page_size`)
- Table component **cannot** be nested inside other components — only at card root
- Table component **cannot** embed other components inside it
- Requires Lark client v7.4+; `lark_md` requires v7.10+; `date` requires v7.6+; `markdown` requires v7.14+

## Example: Simple Data Table

```json
{
  "tag": "table",
  "page_size": 5,
  "row_height": "low",
  "header_style": {
    "text_align": "left",
    "text_size": "normal",
    "background_style": "grey",
    "text_color": "default",
    "bold": true,
    "lines": 1
  },
  "columns": [
    { "name": "task", "display_name": "Task", "data_type": "text", "width": "auto" },
    {
      "name": "status",
      "display_name": "Status",
      "data_type": "options",
      "width": "auto"
    },
    {
      "name": "assignee",
      "display_name": "Assignee",
      "data_type": "persons",
      "width": "auto"
    },
    {
      "name": "due",
      "display_name": "Due Date",
      "data_type": "date",
      "date_format": "YYYY-MM-DD",
      "width": "auto"
    }
  ],
  "rows": [
    {
      "task": "Design review",
      "status": [{ "text": "In Progress", "color": "blue" }],
      "assignee": "ou_xxxx",
      "due": 1699341315000
    },
    {
      "task": "Backend API",
      "status": [{ "text": "Done", "color": "green" }],
      "assignee": ["ou_xxxx", "ou_yyyy"],
      "due": 1699427715000
    }
  ]
}
```
