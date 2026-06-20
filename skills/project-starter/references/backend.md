# Backend Setup Reference

Complete guide for setting up a Node.js backend/API project with TypeScript.

## When to Use This Reference

- Building REST APIs, GraphQL servers, or microservices
- Creating CLI tools or libraries published to npm
- Setting up server-side business logic without a frontend bundler

## Dependencies

Verify current stable versions first:

```bash
for package in lodash-es luxon zod pino typescript @types/node tsx; do
  npm view "$package" version
done
```

Install only the packages the project needs:

```bash
npm install lodash-es luxon zod pino
npm install -D typescript @types/node tsx
```

## TypeScript Configuration

For pure Node.js projects (no Vite):

```json
{
  "extends": ["@tsconfig/node-lts/tsconfig.json", "@tsconfig/strictest/tsconfig.json"],
  "compilerOptions": {
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "verbatimModuleSyntax": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    },
    "resolveJsonModule": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

Key differences from frontend tsconfig:
- `module: "NodeNext"` — Required for Node.js ESM resolution
- `moduleResolution: "NodeNext"` — Follows Node.js ESM import rules strictly
- `outDir` and `rootDir` — TypeScript emits compiled files to `dist/`

## package.json Scripts

```json
{
  "scripts": {
    "dev": "tsx watch src/index.ts",
    "build": "tsc",
    "start": "node dist/index.js",
    "test": "vitest",
    "typecheck": "tsc --noEmit"
  }
}
```

## Entry Point Template

```typescript
// src/index.ts
import { createServer } from 'node:http'

const PORT = process.env.PORT || 3000

const server = createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'application/json' })
  res.end(JSON.stringify({ status: 'ok', timestamp: new Date().toISOString() }))
})

server.listen(PORT, () => {
  console.log(`Server running at http://localhost:${PORT}`)
})
```

For production backends, prefer structured JSON logs over `console.log`. Use `pino` directly or the framework's Pino integration so log records keep machine-readable fields.

## Using Fastify (Recommended)

For production APIs, Fastify is a fast and low-overhead framework:

```bash
npm view fastify version
npm install fastify
```

```typescript
// src/index.ts
import Fastify from 'fastify'

const app = Fastify({ logger: true })

app.get('/health', async () => {
  return { status: 'ok' }
})

app.listen({ port: 3000, host: '0.0.0.0' })
```

## Pino JSON Logging

Use Pino when the backend needs application logs outside a framework logger:

```typescript
// src/logger.ts
import pino from 'pino'

export const logger = pino({
  level: process.env.LOG_LEVEL ?? 'info',
  base: undefined,
})
```

```typescript
import { logger } from './logger.js'

logger.info({ route: '/health' }, 'request handled')
logger.error({ err }, 'request failed')
```

Keep secrets, request bodies, and authorization headers out of logs unless the user explicitly asks for redaction rules and you implement them.

## Zod Schema Validation

```typescript
// src/schemas/user.ts
import { z } from 'zod'

export const UserSchema = z.object({
  id: z.string().uuid(),
  email: z.string().email(),
  name: z.string().min(1).max(100),
  createdAt: z.string().datetime(),
})

export type User = z.infer<typeof UserSchema>

// Usage in route handler:
const result = UserSchema.safeParse(request.body)
if (!result.success) {
  return reply.status(400).send({ errors: result.error.issues })
}
```

## Luxon Date Management

```typescript
import { DateTime } from 'luxon'

const now = DateTime.now().toISO()
const parsed = DateTime.fromISO('2024-01-15T10:00:00Z')
const formatted = DateTime.now().toFormat('yyyy-MM-dd HH:mm:ss')
```

## ESM Import Patterns

Since this is an ESM project, use these patterns:

```typescript
// Named imports (preferred)
import { createServer } from 'node:http'

// Import with file extension (required for NodeNext resolution)
import { config } from './config.js'

// JSON import (requires resolveJsonModule)
import pkg from '../package.json' assert { type: 'json' }

// __dirname equivalent in ESM
import { fileURLToPath } from 'node:url'
import { dirname } from 'node:path'
const __filename = fileURLToPath(import.meta.url)
const __dirname = dirname(__filename)
```

## Publishing as a Library

If building a library to publish to npm:

```json
{
  "name": "my-library",
  "version": "1.0.0",
  "type": "module",
  "main": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "exports": {
    ".": {
      "import": "./dist/index.js",
      "types": "./dist/index.d.ts"
    }
  },
  "files": ["dist"],
  "scripts": {
    "build": "tsc",
    "prepublishOnly": "npm run build"
  }
}
```

## tsconfig for Library Development

```json
{
  "extends": ["@tsconfig/node-lts/tsconfig.json", "@tsconfig/strictest/tsconfig.json"],
  "compilerOptions": {
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "verbatimModuleSyntax": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "outDir": "./dist",
    "rootDir": "./src"
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "**/*.test.ts"]
}
```
