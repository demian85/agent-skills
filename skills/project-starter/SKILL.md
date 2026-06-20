---
name: project-starter
description: Scaffold new projects from scratch using a modern TypeScript tech stack. Use whenever the user wants to create a new project, initialize a codebase, set up project structure, configure tooling, install dependencies, create AGENTS.md project instructions, or bootstrap a frontend/backend/fullstack application. This skill handles package.json, tsconfig, ESLint, Prettier, Vite or Next.js, React, MUI, testing, backend logging, and optional AI stack configuration. Trigger on phrases like "create project", "new project", "scaffold", "bootstrap", "init project", "setup project", "project structure", or whenever generating config files for a new codebase.
---

# Project Starter

Scaffold modern TypeScript projects with a consistent, battle-tested tech stack. This skill generates configuration files, directory structures, and dependency lists for new projects, covering frontend (React + Vite or Next.js), backend (Node.js), fullstack, and optional AI tooling.

## When to Use This Skill

- Creating a new project from scratch
- Initializing a codebase with proper tooling
- Setting up TypeScript, ESLint, Prettier, or Vite configs
- Generating package.json with the predefined stack
- Bootstrapping a React frontend or Node.js backend
- Choosing Next.js when frontend routes need a small backend, server actions, API routes, or secret-bearing inference calls
- Adding testing infrastructure (Vitest) to a new project
- Setting up AI/agent tooling (LangChain, MCP)
- Creating an `AGENTS.md` with project-specific coding-agent instructions

## Core Principles

1. **ESM-first**: All projects use `"type": "module"` in package.json. Every config file is ESM.
2. **TypeScript-native**: tsconfig extends official bases. No `any` suppression. Strict mode enabled.
3. **Standard tooling**: ESLint flat config, Prettier with consistent rules, and Vite or Next.js for frontend runtime.
4. **Minimal overrides**: Use well-known config presets without custom rule overrides. Add rules only when there's a clear reason.
5. **Verify latest stable first**: Treat package names in this skill as recommendations, not pinned install orders. Check the latest stable package versions at scaffold time, then choose exact pins or `^` ranges based on the project's reproducibility needs.
6. **Agent-ready**: Every scaffolded project gets an `AGENTS.md` so future coding agents know the stack, commands, and local constraints.

## Base Stack

Every project starts with these foundations:

| Tool | Purpose | Config File |
|------|---------|-------------|
| Node.js | Runtime | `package.json` (`"type": "module"`) |
| TypeScript | Type system | `tsconfig.json` (extends `@tsconfig/node-lts`) |
| ESLint | Linting | `eslint.config.mjs` (flat config) |
| Prettier | Formatting | `.prettierrc` |
| Vite or Next.js | Frontend runtime | `vite.config.ts` or Next.js app router files |
| AGENTS.md | Agent instructions | `AGENTS.md` |

### Node.js Requirements

- **Minimum**: Match the engines required by the selected framework and verified package versions.
- **Recommended**: Use the current active Node.js LTS unless the target deployment platform requires a different supported LTS.
- All projects use full ESM: `"type": "module"` in package.json

### TypeScript Configuration

Use `@tsconfig/bases` for the foundation. For a Vite + React project:

```json
{
  "extends": ["@tsconfig/vite-react/tsconfig.json", "@tsconfig/strictest/tsconfig.json"],
  "compilerOptions": {
    "verbatimModuleSyntax": true,
    "noEmit": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["src/**/*", "tests/**/*"],
  "exclude": ["node_modules", "dist", "build"]
}
```

For pure Node.js backend projects (no Vite), use:

```json
{
  "extends": ["@tsconfig/node-lts/tsconfig.json", "@tsconfig/strictest/tsconfig.json"],
  "compilerOptions": {
    "verbatimModuleSyntax": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

Key TypeScript settings:
- `module: "ESNext"` / `moduleResolution: "bundler"` (Vite projects)
- `module: "NodeNext"` / `moduleResolution: "NodeNext"` (Node-only projects)
- `verbatimModuleSyntax: true` — forces `import type` for type-only imports
- `noUncheckedIndexedAccess: true` — catches unsafe array/object access
- `exactOptionalPropertyTypes: true` — stricter optional property handling

### ESLint Configuration

Use the flat config format (`eslint.config.mjs`) with these packages:

```javascript
// @ts-check
import eslint from '@eslint/js'
import tseslint from 'typescript-eslint'
import eslintConfigPrettier from 'eslint-config-prettier'
import { defineConfig } from 'eslint/config'

export default defineConfig(
  {
    ignores: [
      'build/',
      'node_modules/',
      'src/**/*.test.ts',
      'src/**/__mocks__/*.ts',
      'src/**/tools/test.ts',
    ],
  },
  eslint.configs.recommended,
  ...tseslint.configs.recommended,
  ...tseslint.configs.recommendedTypeChecked,
  eslintConfigPrettier,
  {
    languageOptions: {
      parserOptions: {
        project: './tsconfig.json',
        tsconfigRootDir: import.meta.dirname,
      },
    },
    rules: {},
  }
)
```

For React projects, add `eslint-plugin-react` and `eslint-plugin-react-hooks` configs before the `eslintConfigPrettier` entry. See [references/frontend.md](references/frontend.md) for the full React-specific ESLint config.

Important: `eslint-config-prettier` MUST be the last entry in the config array to disable conflicting formatting rules.

### Prettier Configuration

Use this `.prettierrc` consistently across all projects:

```json
{
  "tabWidth": 2,
  "useTabs": false,
  "singleQuote": true,
  "semi": false,
  "trailingComma": "es5"
}
```

### Vite Configuration

Standard `vite.config.ts` for React projects:

```typescript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import path from 'node:path'

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
  build: {
    target: 'es2022',
    outDir: 'dist',
    sourcemap: true,
  },
  server: {
    port: 5173,
    open: true,
  },
})
```

For Node.js library/backend projects without a frontend, Vite may not be needed. Use `tsup` or `tsc` directly for building. See [references/backend.md](references/backend.md).

## Frontend Stack

When scaffolding a frontend project, verify the latest stable package versions first, then add the packages that match the chosen app shape.

### Core Framework

| Package | Version Guidance | Purpose |
|---------|------------------|---------|
| `react` | Verify latest stable | UI framework |
| `react-dom` | Verify latest stable | DOM renderer |
| `react-router` | Verify latest stable | Client-side routing for Vite apps |
| `next` | Verify latest stable | Fullstack React framework when routes need backend behavior |

Use Vite + React for static/client-heavy apps that already have an API. Choose Next.js when the frontend needs a small backend, there is no existing API to connect to, routing benefits from server code, or secrets such as AI inference keys must stay off the client. When choosing Next.js, use the `vercel-react-best-practices` skill as the primary implementation guide and this skill only for shared scaffolding concerns like dependency verification, `AGENTS.md`, and baseline tooling.

### State & Data

| Package | Version Guidance | Purpose |
|---------|------------------|---------|
| `@tanstack/react-query` | Verify latest stable | Server state, caching, mutations |

Install alongside the devtools for debugging:
```bash
npm install @tanstack/react-query-devtools
```

### UI Components

| Package | Version Guidance | Purpose |
|---------|------------------|---------|
| `@mui/material` | Verify latest stable | Component library |
| `@mui/icons-material` | Verify latest stable | Material icons |
| `@emotion/react` | Verify latest stable | CSS-in-JS engine |
| `@emotion/styled` | Verify latest stable | Styled components |
| `@fontsource/roboto` | Verify latest stable | Default MUI font |

MUI uses Emotion by default. Import Roboto font weights 300, 400, 500, 700 in your app entry point unless the design system specifies a different font.

### Frontend Directory Structure

```
my-app/
├── public/
│   └── vite.svg
├── src/
│   ├── components/        # Reusable UI components
│   ├── pages/             # Route-level page components
│   ├── hooks/             # Custom React hooks
│   ├── lib/               # Utility functions, API clients
│   ├── queries/           # TanStack Query hooks
│   ├── routes/            # React Router configuration
│   ├── stores/            # State management (if needed beyond React Query)
│   ├── types/             # Shared TypeScript types
│   ├── main.tsx           # Vite entry point
│   └── App.tsx            # Root component
├── tests/                 # Vitest tests
├── index.html
├── vite.config.ts
├── tsconfig.json
├── eslint.config.mjs
├── .prettierrc
├── AGENTS.md
└── package.json
```

For the complete frontend setup guide, see [references/frontend.md](references/frontend.md).

For Next.js projects, see [references/nextjs.md](references/nextjs.md) and then defer to the `vercel-react-best-practices` skill for framework-specific implementation details.

## Backend Stack

When scaffolding a Node.js backend/API project:

### Core Dependencies

| Package | Version Guidance | Purpose |
|---------|------------------|---------|
| `typescript` | Verify latest stable | Type compiler |
| `@types/node` | Verify latest stable for target Node major | Node.js type definitions |

### Recommended Libraries

| Package | Version Guidance | Purpose |
|---------|------------------|---------|
| `lodash-es` | Verify latest stable | Utility functions (ESM build) |
| `luxon` | Verify latest stable | Date/time management |
| `zod` | Verify latest stable | Schema validation, JSON parsing |
| `pino` | Verify latest stable | Structured JSON logging for backend services |

### Backend Directory Structure

```
my-api/
├── src/
│   ├── config/            # Environment/config loading
│   ├── routes/            # API route handlers
│   ├── services/          # Business logic
│   ├── models/            # Data models / schemas
│   ├── middleware/        # Express/Fastify middleware
│   ├── utils/             # Shared utilities
│   ├── types/             # Shared TypeScript types
│   └── index.ts           # Entry point
├── tests/
│   └── unit/
├── dist/                  # Compiled output (gitignored)
├── tsconfig.json
├── eslint.config.mjs
├── .prettierrc
├── AGENTS.md
└── package.json
```

For the complete backend setup guide, see [references/backend.md](references/backend.md).

## AI Stack (Optional)

When the project needs AI/agent capabilities:

| Package | Version Guidance | Purpose |
|---------|------------------|---------|
| `langchain` | Verify latest stable | Agent framework |
| `@langchain/core` | Verify compatible stable | Core abstractions |
| `mcp-use` | Verify latest stable | MCP client/agent integration |

### Critical Peer Dependency Note

All LangChain packages must share the **exact same version** of `@langchain/core`. Pin it explicitly:

```json
{
  "dependencies": {
    "langchain": "<verified-latest-compatible>",
    "@langchain/core": "<verified-compatible-core>",
    "mcp-use": "<verified-latest>"
  }
}
```

Use `~` (tilde) for `@langchain/core` to prevent minor version drift. Run `npm ls @langchain/core` to verify alignment.

For the complete AI stack setup, see [references/ai-stack.md](references/ai-stack.md).

## Testing Stack

Every project should include testing infrastructure:

| Package | Version Guidance | Purpose |
|---------|------------------|---------|
| `vitest` | Verify latest stable | Test runner |
| `@vitest/coverage-v8` | Verify same major/minor as Vitest | Code coverage |
| `happy-dom` or `jsdom` | Verify latest stable | DOM environment for frontend tests |

Vitest config in `vite.config.ts`:

```typescript
export default defineConfig({
  // ...existing config
  test: {
    globals: true,
    environment: 'happy-dom',
    include: ['tests/**/*.{test,spec}.{ts,tsx}'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      exclude: [
        'node_modules/',
        'tests/',
        '**/*.d.ts',
        '**/__mocks__/**',
      ],
    },
  },
})
```

For the complete testing setup guide, see [references/testing.md](references/testing.md).

## Scaffolding Workflow

When asked to scaffold a project, follow this order:

1. **Determine project type**: frontend, backend, or fullstack?
2. **Check for existing files**: Don't overwrite existing configs without confirming.
3. **Choose frontend runtime**: Vite for client apps with an existing API; Next.js when a small backend, server routing, or secret handling is required.
4. **Verify package versions**: Run `npm view <package> version` for every selected package and check Node.js LTS compatibility before writing package.json.
5. **Generate package.json**: Include `"type": "module"` when appropriate, scripts, and dependencies.
6. **Generate config files**: tsconfig, eslint, prettier, vite or Next.js files as needed.
7. **Create directory structure**: Use the templates above.
8. **Create AGENTS.md**: Prefer an available init command or external skill/plugin that generates agent instructions, such as OpenCode or oh-my-openagent. If none is available, create a concise `AGENTS.md` with stack, commands, test expectations, and local constraints.
9. **Add optional stacks**: Frontend deps, AI deps, testing deps based on requirements.
10. **Write a README.md**: Basic project setup instructions (install, dev, build, test).

### Example package.json Scripts

```json
{
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "preview": "vite preview",
    "test": "vitest",
    "test:ui": "vitest --ui",
    "coverage": "vitest run --coverage",
    "lint": "eslint .",
    "lint:fix": "eslint . --fix",
    "format": "prettier --write .",
    "format:check": "prettier --check .",
    "typecheck": "tsc --noEmit"
  }
}
```

For backend-only projects, replace Vite scripts with:
```json
{
  "scripts": {
    "dev": "tsx watch src/index.ts",
    "build": "tsc",
    "start": "node dist/index.js",
    "test": "vitest",
    "lint": "eslint .",
    "format": "prettier --write .",
    "typecheck": "tsc --noEmit"
  }
}
```

## Version Verification

Do not treat any version in this repository as a strict install order. Before scaffolding, verify the current stable release for every package you plan to install:

```bash
npm view typescript version
npm view react version
npm view zod version
```

For a full package set, prefer a small loop so the selected stack is checked in one pass:

```bash
for package in typescript @types/node vite react react-dom react-router zod pino; do
  npm view "$package" version
done
```

Use the live result to decide whether to pin exact versions, use `^` ranges, or use tighter ranges for peer-sensitive packages. For Node.js itself, check the official Node.js download/release page and use the active LTS that matches the deployment target.

### Verified Snapshot

This snapshot was checked on 2026-06-20 with `npm view <package> version` and the Node.js download page. It is evidence that older examples were stale, not a substitute for live verification.

| Package | Verified latest stable |
|---------|------------------------|
| Node.js LTS | 24.17.0 |
| typescript | 6.0.3 |
| @types/node | 26.0.0 |
| vite | 8.0.16 |
| @vitejs/plugin-react | 6.0.2 |
| react | 19.2.7 |
| react-dom | 19.2.7 |
| react-router | 8.0.1 |
| next | 16.2.9 |
| @tanstack/react-query | 5.101.0 |
| @tanstack/react-query-devtools | 5.101.0 |
| @mui/material | 9.1.1 |
| @mui/icons-material | 9.1.1 |
| @emotion/react | 11.14.0 |
| @emotion/styled | 11.14.1 |
| @fontsource/roboto | 5.2.10 |
| eslint | 10.5.0 |
| @eslint/js | 10.0.1 |
| typescript-eslint | 8.61.1 |
| eslint-config-prettier | 10.1.8 |
| eslint-plugin-react | 7.37.5 |
| eslint-plugin-react-hooks | 7.1.1 |
| prettier | 3.8.4 |
| vitest | 4.1.9 |
| @vitest/coverage-v8 | 4.1.9 |
| happy-dom | 20.10.6 |
| jsdom | 29.1.1 |
| @testing-library/react | 16.3.2 |
| @testing-library/jest-dom | 6.9.1 |
| @testing-library/user-event | 14.6.1 |
| lodash-es | 4.18.1 |
| luxon | 3.7.2 |
| zod | 4.4.3 |
| tsx | 4.22.4 |
| fastify | 5.8.5 |
| pino | 10.3.1 |
| langchain | 1.5.0 |
| @langchain/core | 1.2.0 |
| @langchain/openai | 1.5.1 |
| @langchain/langgraph | 1.4.4 |
| mcp-use | 1.32.1 |
| @tsconfig/node-lts | 24.0.0 |
| @tsconfig/vite-react | 8.0.6 |
| @tsconfig/strictest | 2.0.8 |
