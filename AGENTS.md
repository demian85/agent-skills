# PROJECT KNOWLEDGE BASE

Generated: 2026-06-23
Commit: d1738c2
Branch: main

## OVERVIEW

`agent-skills` is a markdown-only [skills.sh](https://skills.sh) repository. It packages installable AI-agent skills plus a public OpenCode/oh-my-openagent config bundle; there is no application runtime, build system, test suite, package manager setup, or root `package.json`.

## STRUCTURE

```text
agent-skills/
|-- AGENTS.md                 # repo-wide agent instructions
|-- README.md                 # human-facing skill table and config bundle docs
|-- skills.sh.json            # skills.sh manifest; paths are relative to each skill dir
|-- skills/
|   |-- project-starter/      # TypeScript scaffolding skill; downstream AGENTS.md rules
|   |-- serverless-framework/ # AWS Serverless Framework reference skill
|   |-- grammy-bot-builder/   # Telegram grammY bot reference skill
|   |-- mui/                  # MUI v9 skill; uses resources/ instead of references/
|   `-- langchain-ts/         # single-file LangChain TypeScript skill
`-- opencode-omo-config/      # public OpenCode and oh-my-openagent JSONC configs
```

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Add a skill | `skills/<name>/SKILL.md`, `skills.sh.json`, `README.md` | Keep all three surfaces in sync. |
| Change trigger behavior | `skills/<name>/SKILL.md` frontmatter and `skills.sh.json` | Descriptions are the primary trigger mechanism. |
| Add deep skill content | `skills/<name>/references/` or `skills/<name>/resources/` | File paths in `skills.sh.json` are relative to the skill directory. |
| Update project scaffolding guidance | `skills/project-starter/` | This is the only skill that explicitly creates downstream `AGENTS.md` files. |
| Update shared OpenCode config | `opencode-omo-config/` and README config table | Config files are public templates, not secret-bearing local config. |
| Validate registry edits | `skills.sh.json` | Run the JSON parse command below. |

## KNOWLEDGE MAP

No active LSP/codegraph code map applies here: the repo has no source modules, exports, or runtime entry points. Treat these content surfaces as the map:

| Surface | Type | Role |
|---------|------|------|
| `skills.sh.json` | manifest | Authoritative skill names, paths, entries, tags, and packaged files. |
| `README.md` | public index | Human-facing skill table, install commands, and shared config description. |
| `skills/*/SKILL.md` | skill entry | YAML frontmatter plus progressive-disclosure instructions. |
| `skills/*/references/` | deep docs | Standard location for longer skill-specific guidance. |
| `skills/mui/resources/` | deep docs | Intentional local deviation from `references/`. |
| `opencode-omo-config/*.jsonc` | config templates | Public model routing, fallback, permissions, and prompt wiring. |

## ADDING OR CHANGING A SKILL

1. Create or edit `skills/<name>/SKILL.md` with frontmatter:
   ```yaml
   ---
   name: skill-name
   description: When to trigger. Be specific and pushy; include user phrases that should activate this skill even if they do not name it explicitly.
   ---
   ```
2. Add supporting files under `skills/<name>/references/` unless the skill already has a reason to use `resources/`.
3. Register every installable file in `skills.sh.json` using `name`, `description`, `path`, `entry`, `tags`, and `files`.
4. Update the README skills table.
5. Validate the manifest:
   ```bash
   node -e "JSON.parse(require('fs').readFileSync('skills.sh.json'))"
   ```

## CONVENTIONS

- `SKILL.md` is the skill entry point. Keep it under about 500 lines and push deep content into referenced markdown files.
- Frontmatter `description` is trigger-critical. Keep it long, specific, synonym-rich, and aligned with `skills.sh.json`.
- Cross-reference deep files from `SKILL.md` with relative paths such as `references/patterns.md`.
- `skills.sh.json` `files` entries are relative to the skill directory, not the repo root.
- Do not add local agent-operation files such as `AGENTS.md` to `skills.sh.json` unless the user explicitly wants them shipped as part of a skill.
- README descriptions should stay short; skill frontmatter and manifest descriptions can be much more trigger-oriented.

## ANTI-PATTERNS

- Do not add `package.json`, dependency management, build tooling, lint tooling, format tooling, or test scripts to this repo.
- Do not invent `npm test`, `npm run build`, `pnpm`, CI, or `.github/workflows` checks.
- Do not treat `opencode-omo-config/` as an installable skills.sh skill.
- Do not store API keys, environment values, or private local config here. The shared configs are public templates.
- Do not leave README and `skills.sh.json` describing different skill sets.
- Do not create nested `AGENTS.md` files in leaf reference/resource folders unless those folders gain their own editing contract.

## COMMANDS

```bash
# Validate the skills.sh manifest after registry edits.
node -e "JSON.parse(require('fs').readFileSync('skills.sh.json'))"

# End-user install examples documented by this repo.
npx skills add <owner/repo> --all
npx skills add <owner/repo> --skill <name>
```

## NOTES

- `.omo/` is OpenCode workspace data and is ignored.
- `.DS_Store` files may be present locally; ignore them for skill structure decisions.
- `skills/langchain-ts/` is currently a single-file skill despite empty-looking support directories in the tree.
- If package version guidance is edited in a skill, verify current stable versions live; do not treat older examples as fixed install order.
