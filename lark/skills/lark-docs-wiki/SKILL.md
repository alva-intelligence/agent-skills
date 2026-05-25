---
name: lark-docs-wiki
description: Lark Docs & Wiki operations — document CRUD, block manipulation, document search, and wiki space/node management. Use when working with Lark documents, document content blocks, or wiki knowledge bases.
metadata:
  version: "1.0.0"
  author: frnd-team
  tags: ["lark", "docs", "docx", "wiki", "knowledge-base", "blocks"]
---

# Lark Docs & Wiki Skill

Use this skill when working with Lark documents (docx), document blocks, searching cloud documents, or managing wiki knowledge base spaces and nodes.

## Available Tools

### Document Tools

| Tool | Purpose | Key Args |
|------|---------|----------|
| `get_document` | Get document metadata (title, revision) | `document_id` |
| `create_document` | Create a new empty document | `title`, `folder_token` |
| `get_document_raw_content` | Get plain-text content of a document | `document_id`, `lang` |

### Block Tools

| Tool | Purpose | Key Args |
|------|---------|----------|
| `list_document_blocks` | Get all blocks in a document (paginated) | `document_id`, `page_size`, `page_token` |
| `get_document_block` | Get a specific block by ID | `document_id`, `block_id` |
| `patch_document_block` | Update a single block | `document_id`, `block_id`, `update_body` |
| `batch_update_document_blocks` | Batch update multiple blocks | `document_id`, `requests_body` |
| `create_block_children` | Create child blocks under a parent | `document_id`, `block_id`, `children`, `index` |
| `get_block_children` | Get all children of a block | `document_id`, `block_id` |
| `batch_delete_block_children` | Delete a range of child blocks | `document_id`, `block_id`, `start_index`, `end_index` |

### Search Tools

| Tool | Purpose | Key Args |
|------|---------|----------|
| `search_documents` | Search cloud docs visible to the user | `search_key`, `count`, `docs_types` |
| `search_wiki_nodes` | Search wiki nodes by keyword | `query`, `space_id`, `node_id` |

### Wiki Space Tools

| Tool | Purpose | Key Args |
|------|---------|----------|
| `list_wiki_spaces` | List all accessible wiki spaces | `page_size`, `page_token` |
| `get_wiki_space` | Get wiki space info | `space_id` |
| `create_wiki_space` | Create a new wiki space | `name`, `description` |
| `get_wiki_space_node` | Get node info by token | `token`, `obj_type` |

### Wiki Node Tools

| Tool | Purpose | Key Args |
|------|---------|----------|
| `list_wiki_nodes` | List child nodes in a wiki space | `space_id`, `parent_node_token` |
| `create_wiki_node` | Create a new node in a wiki space | `space_id`, `obj_type`, `parent_node_token`, `title` |
| `move_wiki_node` | Move a node within/across wiki spaces | `space_id`, `node_token`, `target_parent_token` |
| `update_wiki_node_title` | Update a wiki node title | `space_id`, `node_token`, `title` |
| `move_docs_to_wiki` | Move existing cloud doc into a wiki | `space_id`, `obj_token`, `obj_type` |

## Document Structure

Documents are composed of blocks arranged in a tree structure:

- **Page Block** (block_type=1): The root block. Its `block_id` equals the `document_id`. The document title is the Page block's text content.
- **Text Block** (block_type=2): Regular paragraph text with rich formatting.
- **Heading Blocks** (block_type=3-11): Heading1 through Heading9.
- **Bullet List** (block_type=12): Unordered list item.
- **Ordered List** (block_type=13): Ordered list item.
- **Code Block** (block_type=14): Code with syntax highlighting.
- **Todo Block** (block_type=17): Checkbox item.
- **Bitable Block** (block_type=18): Embedded multidimensional table.
- **Callout Block** (block_type=19): Highlighted container block.
- **File Block** (block_type=23): Attached file.
- **Grid Block** (block_type=24): Column layout container.
- **Grid Column** (block_type=25): A column inside a grid.
- **Image Block** (block_type=27): Embedded image.
- **Sheet Block** (block_type=30): Embedded spreadsheet.
- **Table Block** (block_type=31): Simple table.
- **Table Cell** (block_type=32): Cell inside a table.
- **Quote Container** (block_type=34): Blockquote container.
- **Divider** (block_type=22): Horizontal divider line.

## Workflows

### Read Document Content

1. Use `get_document(document_id)` to get title and revision info
2. Use `get_document_raw_content(document_id)` for a quick plain-text view
3. Use `list_document_blocks(document_id)` for the full structured block tree

### Create and Populate a Document

1. Use `create_document(title="My Doc")` to create an empty document
2. The returned `document_id` is also the Page block's `block_id`
3. Use `create_block_children(document_id, block_id=document_id, children=[...])` to add top-level blocks

### Insert a Text Block

```
create_block_children(
    document_id="...",
    block_id="<parent_block_id>",
    children=[{
        "block_type": 2,
        "text": {
            "elements": [{"text_run": {"content": "Hello world"}}],
            "style": {}
        }
    }]
)
```

### Update Document Title

The title is the Page block's text. Use `patch_document_block`:

```
patch_document_block(
    document_id="...",
    block_id="<document_id>",  # Page block ID = document_id
    update_body={
        "update_text_elements": {
            "elements": [{"text_run": {"content": "New Title"}}]
        }
    }
)
```

### Delete Blocks

Use `batch_delete_block_children(document_id, block_id, start_index, end_index)` to remove children by index range.

### Search for Documents

1. Use `search_documents(search_key="project plan", docs_types="doc,sheet")` to find cloud docs
2. Use `search_wiki_nodes(query="onboarding guide", space_id="...")` to find wiki pages

### Navigate Wiki Structure

1. Use `list_wiki_spaces()` to discover available knowledge bases
2. Use `list_wiki_nodes(space_id)` to browse top-level nodes
3. Use `list_wiki_nodes(space_id, parent_node_token=node_token)` to drill into child nodes
4. Use `get_wiki_space_node(token=node_token)` to get full node details including `obj_token` for document access

### Create Wiki Content

1. Use `create_wiki_space(name="Team Wiki")` to create a new space
2. Use `create_wiki_node(space_id, obj_type="docx", title="Getting Started")` to add a document node
3. Use document block APIs with the returned `obj_token` to populate content

### Move Documents into Wiki

Use `move_docs_to_wiki(space_id, obj_token, obj_type="docx")` to import existing cloud documents into a wiki space.

## Search Best Practices

When using `search_documents` or `search_wiki_nodes`:

- **TOTAL BUDGET**: Maximum 8 search/lookup API calls PER TASK
- **PER-QUERY**: Max 3 search calls for a single piece of information
- Each search MUST use a DIFFERENT strategy (different keywords, not slight variations):
  - ❌ 'CIMB MOM' → 'MOM CIMB' → 'CIMB Minutes' → 'Minutes CIMB' (too similar!)
  - ✅ 'CIMB MOM' → 'CIMB' (broader) → 'Multicurrency Review' (different angle)

**WHEN SEARCH FAILS (2-3 attempts with no results):**
- STOP searching and return results to main agent
- Include: what you searched for, what strategies you tried
- Let the main agent decide next steps

**WHEN YOU ALREADY FOUND IT:**
- If search returned results, do NOT search again with different keywords
- Use what you found

**⚠️ CRITICAL: Handling External Files (type="file")**

When ANY search returns items with `type: "file"` (external files like PDF, uploaded files):
- These are NOT native Lark documents - you CANNOT read them with get_document_raw_content
- The token from search is likely a **wiki node token**, not a Drive file token
- You MUST resolve the token first using `get_wiki_space_node(token='<file_token>')`
- The response will contain `obj_token` - use THIS for Drive download
- Then delegate to the **Drive Agent** with the RESOLVED `obj_token`

Workflow for type="file" results:
1. Get the file token from search result
2. Call `get_wiki_space_node(token='<file_token>')` to resolve to obj_token
3. Check the `obj_type` in response - if it's "file", use obj_token for download
4. Delegate to Drive Agent with the obj_token (not the original search token)

Example response when file type found:
```
Found external file(s) that require Drive access:
1. "Blue Band Core Relaunch.pdf" - wiki_token: ABC123...
   → Resolved obj_token: XYZ789... (using get_wiki_space_node)
   → Delegating to Drive Agent with obj_token: XYZ789...
```

## Important Notes

- `document_id` is always 27 characters long
- The Page block's `block_id` equals the `document_id`
- `create_document` creates an **empty** document — use block APIs to add content
- `search_documents` requires user_access_token (USER_ONLY mode)
- `search_wiki_nodes` requires user_access_token (USER_ONLY mode)
- `create_wiki_space` requires user_access_token (USER_ONLY mode)
- Wiki nodes have two tokens: `node_token` (wiki-specific) and `obj_token` (the actual document token)
- For wiki documents, use `obj_token` with document/block APIs, not the `node_token`
- When getting node info from a wiki URL, the URL token is the `node_token` — use `get_wiki_space_node` to resolve to `obj_token`
- `move_docs_to_wiki` is async — it returns a `task_id` for status polling
- Block operations support `document_revision_id` for version control; use -1 for latest
- `list_document_blocks` returns blocks in pre-order traversal (depth-first)
- Max page sizes: blocks=500, wiki nodes=50, search results=50
- Rate limits: document APIs ~5 req/s per app, block update APIs ~3 req/s per app
