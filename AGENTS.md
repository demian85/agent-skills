# agent-skills

A [skills.sh](https://skills.sh) repository — a collection of markdown-based skills for AI coding assistants. There is no build system, runtime, or tests.

## Adding a Skill

1. Create `skills/<name>/SKILL.md` with frontmatter:
   ```yaml
   ---
   name: skill-name
   description: When to trigger. Be specific and pushy — include user phrases that should activate this skill even if they don't name it explicitly.
   ---
   ```
2. Add reference files under `skills/<name>/references/` (or `resources/`)
3. Register in `skills.sh.json` with: `name`, `description`, `path`, `entry`, `tags`, `files`
4. Update `README.md` skills table
5. Validate JSON: `node -e "JSON.parse(require('fs').readFileSync('skills.sh.json'))"`

## Skill Structure Conventions

- `SKILL.md` is the entry point. Keep it under ~500 lines. Progressive disclosure: metadata → body → references.
- `description` in frontmatter is the primary trigger mechanism. Make it long, specific, and include synonyms.
- Reference files hold deep content. Cross-reference them from SKILL.md with relative paths (e.g., `references/patterns.md`).
- File paths in `skills.sh.json` are relative to the skill directory.
- This repo uses no build tool, no linter, no formatter. Markdown is the only artifact.

## Repo-Specific Notes

- `.omo/` is OpenCode workspace data — already in `.gitignore`
- No `package.json`, no dependency management. Do not add one.
- Skills are installed by end users via `npx skills add <owner/repo> --skill <name>`
