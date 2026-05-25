# Publishing to skills.sh / npx-skills

This repo follows the [skills.sh](https://skills.sh) layout convention so any teammate (or external user) can install skills with one `npx` command. No central registration step required — the `skills` CLI fetches directly from this GitHub repo.

## How users install (already works once repo is public)

```bash
# Install one skill, system-wide (~/.claude/skills/)
npx skills add https://github.com/alva-intelligence/agent-skills --skill frndos/skills/frndos-pr-review-setup

# Install one skill, project-local (./.claude/skills/, writes to skills-lock.json)
npx skills add https://github.com/alva-intelligence/agent-skills \
  --skill frndos/skills/frndos-pr-review-setup \
  --local

# Install all skills in a domain
npx skills add https://github.com/alva-intelligence/agent-skills --path frndos
```

After install, the skill appears in the agent's skill list — invoke via slash command (Claude Code: `/frndos-pr-review-setup`) or by description match.

---

## Required layout convention

The `skills` CLI auto-discovers skills by walking the repo and finding any `**/skills/<skill-name>/SKILL.md` file. That's it. No manifest, no central index.

| Constraint | Why |
|------------|-----|
| Skill folder name matches `SKILL.md` `name:` frontmatter | Used as the install path + slash-command name |
| `SKILL.md` has YAML frontmatter with `name` + `description` | Required for the CLI to recognise it as a skill |
| Description fits on one line (≤ 200 chars) | Used in the skill router prompt |
| References live under `<skill>/references/` | Convention — install copies whole skill folder |
| No binary dependencies in references/ | Skills install as plain files; binaries should be fetched at runtime |

---

## Listing on skills.sh registry (optional)

skills.sh maintains a curated **registry** of public skills. Listing makes the skills discoverable via the skills.sh website search + the `npx skills search` CLI.

### How to list

1. Ensure repo is public.
2. Open a PR against [`skills-sh/registry`](https://github.com/skills-sh/registry) (the index repo).
3. Add an entry to `registry.json`:
   ```json
   {
     "name": "frndos-pr-review-setup",
     "owner": "alva-intelligence",
     "repo": "agent-skills",
     "path": "frndos/skills/frndos-pr-review-setup",
     "description": "Scaffold PR review docs + workflow into any repo (frndOS pipeline).",
     "tags": ["pr-review", "github-actions", "laravel", "nextjs", "fastapi"]
   }
   ```
4. After merge, the skill appears at `https://skills.sh/skills/frndos-pr-review-setup` within ~10 minutes.

**Not required for org use** — teammates can install via direct GitHub URL without registry listing. Listing is for external discoverability.

---

## Versioning

Skills don't have semver — the `skills` CLI installs the latest commit on the default branch by default. Users can pin via:

```bash
npx skills add https://github.com/alva-intelligence/agent-skills@<sha-or-tag> --skill <name>
```

For breaking changes:
1. Tag the last good commit: `git tag stable-<date>` + push.
2. Add a `CHANGELOG.md` entry under the skill folder.
3. Announce on the Lark engineering channel with the tag users should pin to if they want the old behaviour.

---

## Local development

When working on a skill locally without publishing:

```bash
# Symlink into ~/.claude/skills/ for instant testing
ln -s /Users/alva-arhen/Code/work/alva/agent-skills/frndos/skills/frndos-pr-review-setup \
  ~/.claude/skills/frndos-pr-review-setup

# Or use the project-local override
mkdir -p .claude/skills && \
  ln -s /Users/alva-arhen/Code/work/alva/agent-skills/frndos/skills/frndos-pr-review-setup \
  .claude/skills/frndos-pr-review-setup
```

Changes to the skill file take effect on next agent invocation (no restart needed).

---

## Testing a skill before publishing

1. Symlink into `~/.claude/skills/` (see Local development above).
2. Invoke in a scratch directory: `cd /tmp && mkdir test-repo && cd test-repo && git init`.
3. Run the skill: `claude` then type `/frndos-pr-review-setup +all` (or whichever subcommand).
4. Inspect generated files; iterate on the skill until output is correct for a fresh repo.

Run against each known stack profile at least once (clone a sample php-laravel + ts-nextjs + python-fastapi repo, run `+all` in each, eyeball the output).
