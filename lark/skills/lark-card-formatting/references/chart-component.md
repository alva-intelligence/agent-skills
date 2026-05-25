# Chart Component Reference

The chart component (`tag: "chart"`) renders interactive charts based on [VChart](https://www.visactor.io/). Supports line, area, bar, pie, funnel, radar, scatter, word cloud, and progress charts.

## JSON Structure

```json
{
  "tag": "chart",
  "aspect_ratio": "16:9",
  "color_theme": "brand",
  "preview": true,
  "height": "auto",
  "chart_spec": {
    "type": "line",
    "title": { "text": "Chart Title" },
    "data": { "values": [] },
    "xField": "x",
    "yField": "y"
  }
}
```

## Key Fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `aspect_ratio` | String | `16:9` (PC), `1:1` (mobile) | `1:1`, `2:1`, `4:3`, `16:9` |
| `color_theme` | String | `brand` | `brand`, `rainbow`, `complementary`, `converse`, `primary` |
| `preview` | Boolean | `true` | Allow fullscreen/popup view |
| `height` | String | `auto` | `auto` or `[1,999]px` (overrides aspect_ratio) |
| `chart_spec` | Object | — | VChart spec (see types below) |

## Chart Types & Templates

### Line Chart
Trend over time.
```json
{
  "type": "line",
  "title": { "text": "Line Chart" },
  "data": { "values": [
    { "time": "2:00", "value": 8 },
    { "time": "4:00", "value": 9 },
    { "time": "6:00", "value": 11 }
  ]},
  "xField": "time",
  "yField": "value"
}
```

### Area Chart
Like line chart with filled area underneath.
```json
{
  "type": "area",
  "title": { "text": "Area Chart" },
  "data": { "values": [...] },
  "xField": "time",
  "yField": "value"
}
```

### Bar Chart (Vertical)
Compare categories. Add `seriesField` for grouped bars.
```json
{
  "type": "bar",
  "title": { "text": "Bar Chart" },
  "data": { "values": [
    { "type": "A", "year": "2020", "value": 129 },
    { "type": "B", "year": "2020", "value": 22 }
  ]},
  "xField": ["year", "type"],
  "yField": "value",
  "seriesField": "type",
  "legends": { "visible": true, "orient": "bottom" }
}
```

### Horizontal Bar Chart
Same as bar but horizontal.
```json
{
  "type": "bar",
  "direction": "horizontal",
  "title": { "text": "Horizontal Bar" },
  "data": { "values": [
    { "name": "Apple", "value": 214480 },
    { "name": "Google", "value": 155506 }
  ]},
  "xField": "value",
  "yField": "name"
}
```

### Pie Chart
Proportions of a whole.
```json
{
  "type": "pie",
  "title": { "text": "Pie Chart" },
  "data": { "values": [
    { "type": "S1", "value": "340" },
    { "type": "S2", "value": "170" }
  ]},
  "valueField": "value",
  "categoryField": "type",
  "outerRadius": 0.9,
  "label": { "visible": true },
  "legends": { "visible": true, "orient": "right" }
}
```

### Doughnut Chart
Pie with inner radius (hole in center).
```json
{
  "type": "pie",
  "title": { "text": "Doughnut" },
  "data": { "values": [...] },
  "valueField": "value",
  "categoryField": "type",
  "outerRadius": 0.9,
  "innerRadius": 0.3,
  "label": { "visible": true },
  "legends": { "visible": true }
}
```

### Funnel Chart
Stage-by-stage reduction (sales pipeline, conversions).
```json
{
  "type": "funnel",
  "title": { "text": "Funnel" },
  "data": { "values": [
    { "name": "Sent", "value": 5676 },
    { "name": "Viewed", "value": 3872 },
    { "name": "Clicked", "value": 1668 },
    { "name": "Purchased", "value": 565 }
  ]},
  "categoryField": "name",
  "valueField": "value",
  "isTransform": true,
  "label": { "visible": true },
  "transformLabel": { "visible": true }
}
```

### Scatter Chart
Relationship between two variables.
```json
{
  "type": "scatter",
  "title": { "text": "Scatter Plot" },
  "data": { "values": [
    { "x": 24.5, "y": 60, "name": "item1" }
  ]},
  "xField": "x",
  "yField": "y",
  "axes": [
    { "orient": "left", "type": "linear", "title": { "visible": true, "text": "Y Axis" } },
    { "orient": "bottom", "type": "linear", "title": { "visible": true, "text": "X Axis" } }
  ]
}
```

### Radar Chart
Multi-dimensional comparison.
```json
{
  "type": "radar",
  "title": { "text": "Radar" },
  "data": { "values": [
    { "key": "Strength", "value": 5 },
    { "key": "Speed", "value": 4 },
    { "key": "Range", "value": 3 }
  ]},
  "categoryField": "key",
  "valueField": "value",
  "area": { "visible": true },
  "outerRadius": 0.8,
  "axes": [{ "orient": "radius", "label": { "visible": true } }]
}
```

### Combo Chart
Combine bar + line (or other types).
```json
{
  "type": "common",
  "title": { "text": "Combo Chart" },
  "data": [
    { "values": [{ "x": "Mon", "type": "A", "y": 15 }] },
    { "values": [{ "x": "Mon", "type": "B", "y": 22 }] }
  ],
  "series": [
    { "type": "bar", "dataIndex": 0, "seriesField": "type", "xField": ["x", "type"], "yField": "y" },
    { "type": "line", "dataIndex": 1, "seriesField": "type", "xField": "x", "yField": "y" }
  ],
  "axes": [{ "orient": "bottom" }, { "orient": "left" }],
  "legends": { "visible": true, "orient": "bottom" }
}
```

### Linear Progress Bar
```json
{
  "type": "linearProgress",
  "title": { "text": "Progress" },
  "data": { "values": [
    { "type": "Task A", "value": 0.795, "text": "79.5%" },
    { "type": "Task B", "value": 0.25, "text": "25%" }
  ]},
  "direction": "horizontal",
  "xField": "value",
  "yField": "type",
  "seriesField": "type"
}
```

### Circular Progress
```json
{
  "type": "circularProgress",
  "title": { "text": "Circular Progress" },
  "data": { "values": [
    { "type": "A", "value": 0.795, "text": "79.5%" },
    { "type": "B", "value": 0.25, "text": "25%" }
  ]},
  "valueField": "value",
  "categoryField": "type",
  "seriesField": "type",
  "radius": 0.7,
  "innerRadius": 0.4,
  "cornerRadius": 20,
  "indicator": {
    "visible": true,
    "trigger": "hover",
    "title": { "visible": true, "field": "type" },
    "content": [{ "visible": true, "field": "text" }]
  },
  "legends": { "visible": true, "orient": "bottom" }
}
```

### Word Cloud
```json
{
  "type": "wordCloud",
  "title": { "text": "Word Cloud" },
  "data": { "values": [
    { "challenge_name": "keyword1", "sum_count": 128 },
    { "challenge_name": "keyword2", "sum_count": 103 }
  ]},
  "nameField": "challenge_name",
  "valueField": "sum_count",
  "seriesField": "challenge_name"
}
```

## Limits & Notes

- Max **5 chart components** per card recommended
- Chart component does NOT support JavaScript syntax
- Requires Lark client v7.1+
- Mobile does NOT support: texture, conical gradient, grid word cloud layout, extensionMark image repeat
- To disable default media queries (responsive): set `"media": []` in `chart_spec`
- VChart version depends on Lark version (v7.27+ uses VChart 1.12.3)
- Full VChart docs: https://www.visactor.io/vchart/option/barChart
