# opencode-omo-config Knowledge

Generated: 2026-06-23

## OVERVIEW

`opencode-omo-config` is a public, shareable OpenCode and OMO configuration bundle. It is not a skills.sh skill and should not be registered in `skills.sh.json`.

## STRUCTURE

```text
opencode-omo-config/
|-- opencode.jsonc
|-- opencode-go/
|   `-- omo-{low,mid,high}-budget.jsonc
|-- api-access/
|   `-- omo-{low,mid,high}-budget.jsonc
`-- prompts/
    |-- builder.md
    |-- planner.md
    `-- venice-grok-builder.md
```

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Change OpenCode agents or permissions | `opencode.jsonc` | Includes plugin activation, prompts, models, and env-file guardrails. |
| Change OpenCode Go subscription routing | `opencode-go/omo-*-budget.jsonc` | Budget-tier profiles for an active OpenCode Go subscription. |
| Change Venice API-access routing | `api-access/omo-*-budget.jsonc` | Budget-tier profiles that route through Venice instead of OpenCode Go. |
| Change agent prompts | `prompts/*.md` | Referenced by `opencode.jsonc`. |
| Change public docs | `../README.md` | Keep the config table in sync. |

## CONVENTIONS

- Files are JSONC, not strict JSON. Comments and trailing commas are currently part of the style.
- Preserve the public/no-private-data contract from README.
- Keep `.env` and `.env.*` denied while allowing `.env.example` where templates need it.
- Keep prompt file references relative to `opencode.jsonc`, for example `{file:./prompts/builder.md}`.
- When adding a config file, update the README table.
- The `opencode-go/` profiles keep OpenCode Go as the primary provider; Sisyphus is pinned to `opencode-go/kimi-k2.7-code`.
- The `api-access/` profiles replace OpenCode Go models with equivalent `venice/...` models; Sisyphus uses `venice/kimi-k2-7-code`. The low profile is Venice-only (no OpenAI).

## ANTI-PATTERNS

- Do not add API keys, provider secrets, local account identifiers, or private environment values.
- Do not register this directory in `skills.sh.json`.
- Do not silently change model/provider names in the default profile unless the intended routing change is explicit.
