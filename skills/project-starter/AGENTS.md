# project-starter Knowledge

Generated: 2026-06-23

## OVERVIEW

`project-starter` is the scaffolding skill. It owns TypeScript project defaults, framework routing, dependency guidance, and downstream `AGENTS.md` creation.

## STRUCTURE

```text
project-starter/
|-- SKILL.md                 # entry point and shared scaffold contract
`-- references/
    |-- frontend.md          # React/Vite guidance
    |-- nextjs.md            # Next.js routing and secret-bearing routes
    |-- backend.md           # Node backend guidance
    |-- testing.md           # Vitest/testing setup
    `-- ai-stack.md          # LangChain/MCP package alignment
```

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Change base stack | `SKILL.md` | Keep foundations, commands, and AGENTS.md policy aligned. |
| Change frontend defaults | `references/frontend.md` | React/Vite details live there. |
| Add backend-capable frontend guidance | `references/nextjs.md` | Choose Next.js when routes need secrets, server actions, or API routes. |
| Change backend logging/runtime guidance | `references/backend.md` | Keep structured JSON logging guidance intact. |
| Change test setup | `references/testing.md` | This is downstream project guidance, not this repo's test suite. |
| Change AI package guidance | `references/ai-stack.md` | Keep `@langchain/core` alignment explicit. |

## CONVENTIONS

- Every scaffolded project gets an `AGENTS.md` covering stack, commands, tests, environment rules, and local constraints.
- Verify latest stable package versions at scaffold time. Examples are recommendations, not a frozen install order.
- Keep ESM-first defaults: `"type": "module"`, ESM configs, strict TypeScript, and `verbatimModuleSyntax`.
- Do not use `any` suppression in generated TypeScript guidance.
- `eslint-config-prettier` must stay last in ESLint flat config examples.
- Prefer Vite for client-heavy React apps with external APIs; use Next.js when the frontend needs backend behavior or secret-bearing routes.

## ANTI-PATTERNS

- Do not hard-code stale package versions as authoritative.
- Do not move downstream project testing advice into repo-level test expectations.
- Do not hide AGENTS.md creation as optional scaffolding cleanup.
- Do not over-install backend packages; install only what the project needs.
