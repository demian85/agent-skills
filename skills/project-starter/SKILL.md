---
name: project-starter
description: Scaffold new projects from scratch using a predefined modern TypeScript tech stack. Use whenever the user wants to create a new project, initialize a codebase, set up project structure, configure tooling, install dependencies, or bootstrap a frontend/backend application. This skill handles the full setup including package.json, tsconfig, ESLint, Prettier, Vite, React, MUI, testing, and optional AI stack configuration. Trigger on phrases like "create project", "new project", "scaffold", "bootstrap", "init project", "setup project", "project structure", or whenever generating config files for a new codebase.
---

# Project Starter

Scaffold modern TypeScript projects with a consistent, battle-tested tech stack. This skill generates configuration files, directory structures, and dependency lists for new projects — covering both frontend (React + Vite) and backend (Node.js) setups, plus optional AI tooling.

## When to Use This Skill

- Creating a new project from scratch
- Initializing a codebase with proper tooling
- Setting up TypeScript, ESLint, Prettier, or Vite configs
- Generating package.json with the predefined stack
- Bootstrapping a React frontend or Node.js backend
- Adding testing infrastructure (Vitest) to a new project
- Setting up AI/agent tooling (LangChain, MCP)

## Core Principles

1. **ESM-first**: All projects use `"type": "module"` in package.json. Every config file is ESM.
2. **TypeScript-native**: tsconfig extends official bases. No `any` suppression. Strict mode enabled.
3. **Standard tooling**: ESLint flat config, Prettier with consistent rules, Vite for bundling.
4. **Minimal overrides**: Use well-known config presets without custom rule overrides. Add rules only when there's a clear reason.
5. **Latest stable**: Pin to latest stable versions at time of scaffolding. Use `^` ranges in package.json for flexibility.

## Base Stack

Every project starts with these foundations:

| Tool | Purpose | Config File |
|------|---------|-------------|
| Node.js | Runtime | `package.json` (`"type": "module"`) |
| TypeScript | Type system | `tsconfig.json` (extends `@tsconfig/node-lts`) |
| ESLint | Linting | `eslint.config.mjs` (flat config) |
| Prettier | Formatting | `.prettierrc` |
| Vite | Bundler/dev server | `vite.config.ts` |

### Node.js Requirements

- **Minimum**: Node.js 20.19.0 (supports `require(esm)`)
- **Recommended**: Node.js 22.12.0+ LTS or 24.x
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

When scaffolding a frontend project, add these dependencies:

### Core Framework

| Package | Version Range | Purpose |
|---------|--------------|---------|
| `react` | `^19.0.0` | UI framework |
| `react-dom` | `^19.0.0` | DOM renderer |
| `react-router` | `^7.0.0` | Routing (all-in-one, replaces react-router-dom) |

React 19 is ESM-native. React Router v7 merges `react-router-dom` into the main `react-router` package.

### State & Data

| Package | Version Range | Purpose |
|---------|--------------|---------|
| `@tanstack/react-query` | `^5.0.0` | Server state, caching, mutations |

Install alongside the devtools for debugging:
```bash
npm install @tanstack/react-query-devtools
```

### UI Components

| Package | Version Range | Purpose |
|---------|--------------|---------|
| `@mui/material` | `^9.0.0` | Component library |
| `@mui/icons-material` | `^9.0.0` | Material icons |
| `@emotion/react` | `^11.0.0` | CSS-in-JS engine |
| `@emotion/styled` | `^11.0.0` | Styled components |
| `@fontsource/roboto` | latest | Default MUI font |

MUI v9 uses Emotion by default. Import Roboto font weights 300, 400, 500, 700 in your app entry point.

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
└── package.json
```

For the complete frontend setup guide, see [references/frontend.md](references/frontend.md).

## Backend Stack

When scaffolding a Node.js backend/API project:

### Core Dependencies

| Package | Version Range | Purpose |
|---------|--------------|---------|
| `typescript` | `^6.0.0` | Type compiler |
| `@types/node` | `^22.0.0` | Node.js type definitions |

### Recommended Libraries

| Package | Version Range | Purpose |
|---------|--------------|---------|
| `lodash-es` | `^4.0.0` | Utility functions (ESM build) |
| `luxon` | `^3.0.0` | Date/time management |
| `zod` | `^3.0.0` | Schema validation, JSON parsing |

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
└── package.json
```

For the complete backend setup guide, see [references/backend.md](references/backend.md).

## AI Stack (Optional)

When the project needs AI/agent capabilities:

| Package | Version Range | Purpose |
|---------|--------------|---------|
| `langchain` | `^1.0.0` | Agent framework |
| `@langchain/core` | `^1.0.0` | Core abstractions |
| `mcp-use` | `^1.0.0` | MCP client/agent integration |

### Critical Peer Dependency Note

All LangChain packages must share the **exact same version** of `@langchain/core`. Pin it explicitly:

```json
{
  "dependencies": {
    "langchain": "^1.4.0",
    "@langchain/core": "~1.1.0",
    "mcp-use": "^1.28.0"
  }
}
```

Use `~` (tilde) for `@langchain/core` to prevent minor version drift. Run `npm ls @langchain/core` to verify alignment.

For the complete AI stack setup, see [references/ai-stack.md](references/ai-stack.md).

## Testing Stack

Every project should include testing infrastructure:

| Package | Version Range | Purpose |
|---------|--------------|---------|
| `vitest` | `^4.0.0` | Test runner (Vite-native) |
| `@vitest/coverage-v8` | `^4.0.0` | Code coverage |
| `happy-dom` or `jsdom` | latest | DOM environment for frontend tests |

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
3. **Generate package.json**: Include `"type": "module"`, scripts, and dependencies.
4. **Generate config files**: tsconfig, eslint, prettier, vite (if frontend).
5. **Create directory structure**: Use the templates above.
6. **Add optional stacks**: Frontend deps, AI deps, testing deps based on requirements.
7. **Write a README.md**: Basic project setup instructions (install, dev, build, test).

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

## Version Reference

These are the latest stable versions at the time of this skill's creation. Use `^` ranges in package.json to allow patch/minor updates:

| Package | Latest Stable |
|---------|--------------|
| typescript | ^6.0.0 |
| @types/node | ^22.0.0 |
| vite | ^8.0.0 |
| react | ^19.0.0 |
| react-dom | ^19.0.0 |
| react-router | ^7.0.0 |
| @tanstack/react-query | ^5.0.0 |
| @mui/material | ^9.0.0 |
| @mui/icons-material | ^9.0.0 |
| eslint | ^10.0.0 |
| typescript-eslint | ^8.0.0 |
| prettier | ^3.0.0 |
| vitest | ^4.0.0 |
| lodash-es | ^4.0.0 |
| luxon | ^3.0.0 |
| zod | ^3.0.0 |
| langchain | ^1.0.0 |
| @langchain/core | ~1.1.0 |
| mcp-use | ^1.0.0 |

Always verify the latest version with `npm view <package> version` before scaffolding if the project has specific version requirements.
