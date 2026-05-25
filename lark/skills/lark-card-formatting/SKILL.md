---
name: lark-card-formatting
description: Lark Card response formatting guidelines for the FRnD bot. Covers general markdown rules, normal rich text cards, and collapsible panel (rich card) layouts. Load this skill BEFORE writing your final response.
metadata:
  version: "1.0.0"
  author: frnd-team
  tags:
    ["lark", "card", "formatting", "markdown", "rich-text", "collapsible-panel"]
---

# Lark Card Formatting Skill

**LOAD THIS SKILL before writing your final response.** It defines how to format
output so the Lark Card renderer produces clean, readable cards.

Your response is rendered inside a Lark Card JSON 2.0 `{"tag": "markdown"}` component.
The card builder handles the JSON wrapper — you only produce the **markdown content**.

---

## General Rules (Apply to ALL responses)

### G1 — Headers Rule

NEVER use `#`, `##`, `###` markdown headers.
Start With `####`, `#####`, `######`, etc.

```
WRONG:  ## Features
WRONG:  ### Features
RIGHT:  #### Features
```

### G2 — Line Breaks

Use `<br>` for line breaks within the card markdown content.
For visual separation, use `<br>` tags.
ALWAYS add A newline (`<br>`) before a bold section title and after bullet/list blocks.
use <hr> for visual horizontal line separation between sections.

```
WRONG:
**Next Section**
lorem ipsum dolor sit amet
**Next Section**

RIGHT:
<br>
**Next Section**
lorem ipsum dolor sit amet
<hr>
**Next Section**
```

### G3 — Separators After Lists

After a bullet/list block ends, ALWAYS insert `<hr>` (horizontal rule) before new text.
This creates clean visual breaks between sections.

```
WRONG:
- item 1
- item 2
**Next Section**

RIGHT:
- item 1
- item 2
<hr>

**Next Section**
```

### G4 — Empty Lines Between Blocks

Always add an empty line (<br>) between:

- A table and surrounding text
- A code block and surrounding text
- A bold title and its content
- Distinct content sections

### G5 — Table Limits

- Maximum **3 markdown tables** per response
- Markdown tables in the rich text component show max **5 data rows** per page (auto-paginated)
- Maximum **4 tables** per single markdown component
- If more data is needed: combine into one table, use bullet lists, or offer to create a Lark Sheet/Doc

### G6 — Visual Enhancements

| Element      | Usage                                         | Example                              |
| ------------ | --------------------------------------------- | ------------------------------------ |
| Emojis       | Categorize content (1-2 per section)          | categorize content, status, insights |
| Tables       | Schedules, comparisons, multi-column data     | `\| Col1 \| Col2 \|`                 |
| Separators   | Clean breaks between major sections           | `<hr>`                               |
| `**Bold**`   | Key terms, section labels                     | `**Important**`                      |
| `` `code` `` | Technical terms, commands, file paths         | `` `GET /api/v1` ``                  |
| `*Italic*`   | Subtle emphasis (use sparingly)               | `*note*`                             |
| Lists        | Numbered for sequential, bullets for features | `1.` or `-`                          |

### G7 — Text Coloring (use sparingly)

Lark markdown supports colored text via `<font>` tags:

```
<font color='red'>Error text</font>
<font color='green'>Success text</font>
<font color='grey'>Muted text</font>
```

### G8 — @Mentions

To mention a user in the card: `<at id=OPEN_ID></at>`

- No quotes around the id value
- No text between tags — Lark resolves the name automatically
- To mention everyone: `<at id=all></at>`

### G9 — Code Blocks

Use fenced code blocks with language specification:

````
```python
def hello():
    print("world")
```
````

Supported languages: python, json, javascript, typescript, bash, sql, go, java, etc.

### G10 — Blockquotes

Use `>` for quoted text:

```
> This is a blockquote
> Second line of the quote
```

### G11 — Tags

Use `<text_tag>` for inline colored tags/labels:

```
<text_tag color='green'>Completed</text_tag>
<text_tag color='red'>Critical</text_tag>
<text_tag color='blue'>In Progress</text_tag>
```

Supported colors: neutral, blue, turquoise, lime, orange, violet, indigo, wathet, green, yellow, red, purple, carmine.

### G12 — Links

```
[Display Text](https://example.com)
```

Links must include `http://` or `https://` schema.

---

## Response Type 1: Normal Rich Text Card

**USE WHEN:** Normal conversational chat, greetings, explanations, Q&A,
responses that don't need complex section separation.

**FORMAT:** Write your response as plain markdown following the General Rules above.
The card builder wraps it in a single `{"tag": "markdown"}` component automatically.

**Example response:**

```
**Welcome to FRnD!**

I can help you with:
- Calendar management and scheduling
- Document search and analysis
- Task tracking and reminders
- Data analysis from Bitable

<hr>

**Quick Tips**
- Mention me in any group chat to get started
- Use `/help` for available commands
- I remember context from our conversations
```

**Key principles for normal cards:**

1. Keep it concise and scannable
2. Use **bold** for emphasis, never headers
3. Use bullet lists for multiple items
4. Add `<hr>` separators between logical sections
5. Use emojis sparingly for visual landmarks

---

## Response Type 2: Rich Card with Collapsible Panels

**USE WHEN:** Complex/structured responses that need section separation:

- Long reports, summaries, or analysis
- Multiple distinct sections of content
- Data-heavy responses with different categories
- Comparison or evaluation results

**FORMAT:** Use the `<<<RICH_CARD>>>` tag syntax. The card builder parses this
into collapsible panels automatically.

```
<<<RICH_CARD>>>
<<<TITLE color="blue">>>Card Title Here<<</TITLE>>>
<<<SECTION title="First Section" expanded="true">>>
Content for the first section using markdown format.
- Bullet points work here
- **Bold** for emphasis

| Column 1 | Column 2 |
|----------|----------|
| Data A   | Data B   |
<<</SECTION>>>
<<<SECTION title="Second Section">>>
More content here. This section is collapsed by default.
<<</SECTION>>>
<<<SECTION title="Third Section">>>
Additional content if needed.
<<</SECTION>>>
<<</RICH_CARD>>>
```

**Rich Card Rules:**

1. First section: `expanded="true"`, rest default to collapsed
2. Minimum 2 sections required
3. Each tag MUST have a closing tag
4. Always exactly 3 angle brackets: `<<<` and `>>>` (never `<<` or `>>`)
5. All General Rules (G1-G12) apply inside sections too

**Color options for TITLE:**

- `blue` — analysis, information
- `green` — summary, success
- `purple` — report, documentation
- `orange` — findings, warnings
- `red` — errors, critical issues
- `yellow` — notes, caution

**FATAL ERRORS (card silently breaks):**

```
WRONG: <<<SECTION>>> to close (missing /)
WRONG: <<< /TITLE >>> (spaces around /)
WRONG: <<</ SECTION>>> (space after /)

RIGHT: <<</SECTION>>>
RIGHT: <<</TITLE>>>
RIGHT: <<</RICH_CARD>>>
```

---

## Decision Guide: Which Response Type?

| Scenario                          | Type      | Why                            |
| --------------------------------- | --------- | ------------------------------ |
| "Hi, how are you?"                | Normal    | Simple greeting                |
| "What can you do?"                | Normal    | Brief explanation              |
| "Summarize this document"         | Rich Card | Long content, multiple aspects |
| "Compare option A vs B"           | Rich Card | Structured comparison          |
| "Analyze this data"               | Rich Card | Multiple findings/sections     |
| "Create an event tomorrow"        | Normal    | Action confirmation            |
| "What meetings do I have?"        | Normal    | Simple list response           |
| "Generate a weekly report"        | Rich Card | Multi-section report           |
| "What's the status of project X?" | Rich Card | Multiple dimensions            |
| "Thanks!"                         | Normal    | Simple acknowledgment          |

---

## Additional References

For specialized formatting needs, load the following references:

- `get_skill_reference("lark-card-formatting", "table-component.md")` — Full table component docs with column types, data formats, pagination
- `get_skill_reference("lark-card-formatting", "chart-component.md")` — VChart-based chart component (line, bar, pie, radar, funnel, etc.)
- `get_skill_reference("lark-card-formatting", "collapsible-panel.md")` — Detailed collapsible panel JSON structure and nesting rules
