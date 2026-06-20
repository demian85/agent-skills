# AI Stack Reference

Complete guide for setting up LangChain, MCP, and related AI tooling.

## When to Use This Reference

- Building AI agents or LLM-powered applications
- Integrating with MCP (Model Context Protocol) servers
- Creating chatbots, RAG pipelines, or autonomous agents

## Dependencies

Verify current stable versions first:

```bash
for package in langchain @langchain/core @langchain/openai @langchain/langgraph mcp-use zod; do
  npm view "$package" version
done
```

Then install the compatible set the project needs:

```bash
npm install langchain @langchain/core @langchain/openai @langchain/langgraph mcp-use zod
```

## Critical: @langchain/core Version Alignment

All LangChain packages transitively depend on `@langchain/core`. They must share the **exact same version**:

```json
{
  "dependencies": {
    "langchain": "<verified-latest-compatible>",
    "@langchain/core": "<verified-compatible-core>",
    "@langchain/openai": "<verified-latest-compatible>"
  },
  "overrides": {
    "@langchain/core": "<verified-compatible-core>"
  }
}
```

Use `~` (tilde) for `@langchain/core` to prevent automatic minor version bumps. Always run `npm ls @langchain/core` after install to verify all packages resolve to the same version.

## Basic LangChain Agent Setup

```typescript
// src/agent/index.ts
import { ChatOpenAI } from '@langchain/openai'
import { createReactAgent } from '@langchain/langgraph/prebuilt'
import { tool } from '@langchain/core/tools'
import { z } from 'zod'

const weatherTool = tool(
  async ({ city }) => {
    // Implementation here
    return `Weather in ${city}: sunny, 72°F`
  },
  {
    name: 'get_weather',
    description: 'Get current weather for a city',
    schema: z.object({ city: z.string() }),
  }
)

const model = new ChatOpenAI({
  model: 'gpt-4o',
  temperature: 0,
})

export const agent = createReactAgent({
  llm: model,
  tools: [weatherTool],
})
```

## MCP (Model Context Protocol) Integration

`mcp-use` provides a client for connecting to MCP servers:

```typescript
// src/mcp/client.ts
import { MCPClient } from 'mcp-use'

const client = new MCPClient({
  serverUrl: 'http://localhost:3001/sse',
})

export async function listTools() {
  await client.connect()
  const tools = await client.listTools()
  return tools
}

export async function callTool(name: string, args: unknown) {
  return client.callTool(name, args)
}
```

## Streaming Responses

```typescript
import { HumanMessage } from '@langchain/core/messages'

const stream = await agent.stream(
  { messages: [new HumanMessage('What is the weather in Tokyo?')] },
  { streamMode: 'values' }
)

for await (const chunk of stream) {
  console.log(chunk)
}
```

## Environment Variables

AI projects typically need these environment variables:

```bash
# .env
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...
MCP_SERVER_URL=http://localhost:3001
```

Load them with a type-safe config:

```typescript
// src/config/env.ts
import { z } from 'zod'

const envSchema = z.object({
  OPENAI_API_KEY: z.string().min(1),
  ANTHROPIC_API_KEY: z.string().optional(),
  MCP_SERVER_URL: z.string().url().default('http://localhost:3001'),
  NODE_ENV: z.enum(['development', 'production', 'test']).default('development'),
})

export const env = envSchema.parse(process.env)
```

## Type-Safe Tool Definitions

```typescript
import { tool } from '@langchain/core/tools'
import { z } from 'zod'

export const searchDocuments = tool(
  async ({ query, limit }) => {
    // Implementation
    return { results: [] }
  },
  {
    name: 'search_documents',
    description: 'Search internal documents by keyword',
    schema: z.object({
      query: z.string().describe('Search query string'),
      limit: z.number().int().min(1).max(50).default(10),
    }),
  }
)
```

## Project Structure for AI Projects

```
my-ai-project/
├── src/
│   ├── agent/             # Agent definitions
│   │   └── index.ts
│   ├── tools/             # LangChain tools
│   │   ├── weather.ts
│   │   └── search.ts
│   ├── mcp/               # MCP client/servers
│   │   ├── client.ts
│   │   └── server.ts
│   ├── chains/            # Pre-built LLM chains
│   ├── prompts/           # Prompt templates
│   ├── config/
│   │   └── env.ts
│   └── index.ts
├── tests/
├── .env
├── .env.example
└── package.json
```
