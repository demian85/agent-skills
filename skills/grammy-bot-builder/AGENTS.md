# grammy-bot-builder Knowledge

Generated: 2026-06-23

## OVERVIEW

`grammy-bot-builder` is the Telegram bot skill for grammY on TypeScript, Node.js, Bun, and Deno. It owns bot setup, API usage, plugins, middleware patterns, webhooks, sessions, conversations, and hosting guidance.

## STRUCTURE

```text
grammy-bot-builder/
|-- SKILL.md
`-- references/
    |-- core-api.md
    |-- plugins.md
    `-- patterns.md
```

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Change trigger phrases or quick start | `SKILL.md` | Keep Telegram, grammY, bot commands, webhook, session, and conversation triggers explicit. |
| Edit Bot API/context/filter details | `references/core-api.md` | Core grammY API surface. |
| Edit official plugin guidance | `references/plugins.md` | Sessions, menus, conversations, files, hydrate, runner, and related plugins. |
| Edit project structure or pitfalls | `references/patterns.md` | Middleware order, file handling, hosting, and error patterns. |

## CONVENTIONS

- Keep examples type-safe and aligned with grammY's middleware model.
- Middleware order matters; preserve ordering guidance when moving examples.
- For local files, use `new InputFile()` patterns rather than implying raw filesystem paths are valid uploads.
- Keep Node.js, Bun, and Deno examples clearly labeled so runtime-specific syntax does not bleed across sections.

## ANTI-PATTERNS

- Do not collapse plugin material into `SKILL.md`; keep the entry point navigable.
- Do not present Telegram bot tokens or environment values as committed config.
- Do not remove webhook/long-polling distinctions when shortening guidance.
