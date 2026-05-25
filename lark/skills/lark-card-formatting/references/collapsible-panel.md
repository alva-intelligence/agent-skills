# Collapsible Panel Component Reference

The collapsible panel allows folding secondary information (notes, longer texts) within a card to highlight primary information.

## JSON 2.0 Structure

```json
{
  "tag": "collapsible_panel",
  "expanded": false,
  "header": {
    "title": {
      "tag": "markdown",
      "content": "**Panel Title**"
    },
    "background_color": "grey",
    "vertical_align": "center",
    "padding": "4px 0px 4px 8px",
    "width": "auto",
    "icon": {
      "tag": "standard_icon",
      "token": "down-small-ccm_outlined",
      "color": "",
      "size": "16px 16px"
    },
    "icon_position": "right",
    "icon_expanded_angle": -180
  },
  "border": {
    "color": "grey",
    "corner_radius": "5px"
  },
  "direction": "vertical",
  "vertical_spacing": "8px",
  "padding": "8px 8px 8px 8px",
  "margin": "0px 0px 0px 0px",
  "elements": [
    {
      "tag": "markdown",
      "content": "Panel body content in markdown"
    }
  ]
}
```

## Key Fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `expanded` | Boolean | `false` | Whether panel is expanded on load |
| `header.title.tag` | String | — | `plain_text` or `markdown` |
| `header.title.content` | String | — | Title text |
| `header.background_color` | String | transparent | Title area background color |
| `header.width` | String | `fill` | `fill`, `auto`, or `auto_when_fold` |
| `header.icon_position` | String | `right` | `left`, `right`, or `follow_text` |
| `header.icon_expanded_angle` | Number | `180` | Rotation angle when expanded (-180, -90, 90, 180) |
| `border.color` | String | — | Border color (see color enums) |
| `border.corner_radius` | String | `5px` | Corner radius |
| `direction` | String | `vertical` | `vertical` or `horizontal` |
| `vertical_spacing` | String | `12px` | `small`(4px), `medium`(8px), `large`(12px), `extra_large`(16px), or `[0,99]px` |
| `horizontal_spacing` | String | `8px` | Same options as vertical_spacing |
| `padding` | String | `0px` | Container padding, range [-99,99]px |
| `margin` | String | `0px` | Container margin, range [-99,99]px |
| `background_color` | String | transparent | Panel background color |

## Nesting Rules

- Max **5 layers** of nested components (avoid deep nesting)
- **Cannot** embed form components
- Can embed markdown, images, tables, and other content components

## Styles

### Default Collapsed (with border)
```json
{
  "tag": "collapsible_panel",
  "expanded": false,
  "header": {
    "title": { "tag": "plain_text", "content": "Click to expand" },
    "vertical_align": "center",
    "icon": { "tag": "standard_icon", "token": "down-small-ccm_outlined", "size": "16px 16px" },
    "icon_position": "right",
    "icon_expanded_angle": -180
  },
  "border": { "color": "grey", "corner_radius": "5px" },
  "padding": "8px 8px 8px 8px",
  "elements": [{ "tag": "markdown", "content": "Hidden content here" }]
}
```

### Expanded with Colored Header
```json
{
  "tag": "collapsible_panel",
  "expanded": true,
  "header": {
    "title": { "tag": "markdown", "content": "**Section Title**" },
    "background_color": "blue",
    "vertical_align": "center",
    "icon": { "tag": "standard_icon", "token": "down-small-ccm_outlined", "color": "white", "size": "16px 16px" },
    "icon_position": "right",
    "icon_expanded_angle": -180
  },
  "border": { "color": "grey", "corner_radius": "5px" },
  "padding": "8px 8px 8px 8px",
  "elements": [{ "tag": "markdown", "content": "Visible content here" }]
}
```

### Borderless with Follow-Text Icon
```json
{
  "tag": "collapsible_panel",
  "expanded": true,
  "header": {
    "title": { "tag": "markdown", "content": "**Title**" },
    "width": "auto_when_fold",
    "vertical_align": "center",
    "padding": "4px 0px 4px 8px",
    "icon": { "tag": "standard_icon", "token": "down-small-ccm_outlined", "size": "16px 16px" },
    "icon_position": "follow_text",
    "icon_expanded_angle": -180
  },
  "padding": "8px 8px 8px 8px",
  "elements": [{ "tag": "markdown", "content": "Content without border" }]
}
```

## Color Enumeration

Available colors for `background_color` and `border.color`:
`default`, `blue`, `turquoise`, `lime`, `orange`, `violet`, `indigo`, `wathet`, `green`, `yellow`, `red`, `purple`, `carmine`, `grey`, `neutral`
