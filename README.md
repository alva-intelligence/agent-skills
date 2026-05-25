# agent-skills

Reusable agent skills + agents for alva-intelligence repos. Installable into any Claude Code / Cursor / OpenCode workspace via [`skills.sh`](https://skills.sh) / `npx skills`.

## Layout

```
agent-skills/
├── frndos/                          # frndOS-specific tooling
│   ├── skills/                      # invokable skills (slash commands)
│   │   ├── frndos-pr-review-setup/
│   │   └── tiktok-social-analyzer/
│   └── agents/                      # subagent definitions
├── lark/                            # Lark Open Platform skills
│   ├── skills/
│   │   ├── lark-bitable/
│   │   ├── lark-calendar/
│   │   ├── lark-card-formatting/
│   │   ├── lark-docs-wiki/
│   │   ├── lark-drive/
│   │   ├── lark-email/
│   │   ├── lark-im-contacts/
│   │   ├── lark-sheets/
│   │   └── lark-tasks/
│   └── agents/
└── <future-domain>/                 # additional domains added here
    ├── skills/
    └── agents/
```

Each top-level folder is a **domain** — a logical grouping of related skills + agents. New domains sit alongside `frndos/`.

## Skills catalogue

### frndos/

| Skill | Purpose |
|-------|---------|
| `frndos-pr-review-setup` | Scaffold `REVIEW.md`, `KNOWLEDGE.md`, `.github/workflows/review.yml` into any repo so it joins the central PR-review pipeline. |
| `tiktok-social-analyzer` | TikTok social-data analysis primitives — content, engagement, trends. |

### lark/

Lark Open Platform skills (Bitable, Calendar, Card formatting, Docs/Wiki, Drive, Email, IM/Contacts, Sheets, Tasks). See [`lark/README.md`](./lark/README.md) for full list.

## Install

### Single skill

```bash
npx skills add https://github.com/alva-intelligence/agent-skills --skill frndos/skills/frndos-pr-review-setup
```

### Whole domain

```bash
npx skills add https://github.com/alva-intelligence/agent-skills --path frndos
```

### Manual (Claude Code)

Clone into `.claude/skills/` (project-level) or `~/.claude/skills/` (user-level):

```bash
git clone https://github.com/alva-intelligence/agent-skills ~/.claude/agent-skills
ln -s ~/.claude/agent-skills/frndos/skills/frndos-pr-review-setup ~/.claude/skills/
```

## Adding a new skill

1. Pick (or create) a domain folder.
2. Create `<domain>/skills/<skill-name>/SKILL.md` with frontmatter:
   ```markdown
   ---
   name: <skill-name>
   description: <one-line; used by skill router>
   ---
   ```
3. Add `references/` subdir for templates, longer docs.
4. Add a row to the catalogue table above.
5. Open a PR.

## Contributing

Skills must be:
- **Self-contained** — no hard dependencies on other repos at runtime.
- **Idempotent** — re-running must not corrupt prior state; ask before overwriting.
- **Agnostic where possible** — templates parametrise stack/branch/runner details rather than hardcoding.
- **Documented** — `SKILL.md` lists every subcommand, every prompt, every file written.

## License

MIT — see [LICENSE](./LICENSE).
