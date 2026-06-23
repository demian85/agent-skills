# opencode-omo-config Knowledge

Generated: 2026-06-23

## OVERVIEW

`opencode-omo-config` is a public, shareable OpenCode and oh-my-openagent configuration bundle. It is not a skills.sh skill and should not be registered in `skills.sh.json`.

## STRUCTURE

```text
opencode-omo-config/
|-- opencode.jsonc
|-- oh-my-openagent.jsonc
|-- oh-my-openagent-low-budget.jsonc
|-- oh-my-openagent-mid-budget.jsonc
|-- oh-my-openagent-high-budget.jsonc
`-- prompts/
    |-- builder.md
    |-- planner.md
    `-- venice-grok-builder.md
```

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Change OpenCode agents or permissions | `opencode.jsonc` | Includes plugin activation, prompts, models, and env-file guardrails. |
| Change default model routing | `oh-my-openagent.jsonc` | Main profile. |
| Change budget profiles | `oh-my-openagent-*-budget.jsonc` | Keep low/mid/high intent clear. |
| Change agent prompts | `prompts/*.md` | Referenced by `opencode.jsonc`. |
| Change public docs | `../README.md` | Keep the config table in sync. |

## CONVENTIONS

- Files are JSONC, not strict JSON. Comments and trailing commas are currently part of the style.
- Preserve the public/no-private-data contract from README.
- Keep `.env` and `.env.*` denied while allowing `.env.example` where templates need it.
- Keep prompt file references relative to `opencode.jsonc`, for example `{file:./prompts/builder.md}`.
- When adding a config file, update the README table.

## ANTI-PATTERNS

- Do not add API keys, provider secrets, local account identifiers, or private environment values.
- Do not register this directory in `skills.sh.json`.
- Do not silently change model/provider names across all budget profiles unless the intended routing change is explicit.
