---
name: langchain-ts
description: Comprehensive guide for building AI agents with LangChain TypeScript SDK. Covers LangChain agents, LangGraph orchestration, Deep Agents harness, tools, models, middleware, memory, streaming, and provider integrations. Use when working with langchain npm packages, creating ReAct agents, building stateful workflows with StateGraph, implementing tool-calling loops, or integrating LLM providers (OpenAI, Anthropic, Google, etc.) in TypeScript/JavaScript.
---

# LangChain TypeScript SDK

## Overview

LangChain JS/TS is an open-source framework for building LLM-powered agents and applications. It provides:

- **LangChain**: High-level agent framework (`langchain` package) with `createAgent()`, tools, models, and middleware
- **LangGraph**: Low-level orchestration runtime (`@langchain/langgraph`) for stateful, durable execution
- **Deep Agents**: "Batteries-included" agent harness (`deepagents` package) with planning, filesystem, and subagents
- **LangSmith**: Observability platform for tracing and evaluation

## Architecture

```
┌─────────────────────────────────────────┐
│           Deep Agents SDK               │  ← Agent harness (planning, filesystem, subagents)
├─────────────────────────────────────────┤
│           LangChain Framework           │  ← createAgent(), tools, middleware, models
├─────────────────────────────────────────┤
│           LangGraph Runtime             │  ← StateGraph, persistence, streaming, interrupts
├─────────────────────────────────────────┤
│      @langchain/core Abstractions       │  ← Messages, Runnable interface, BaseChatModel
├─────────────────────────────────────────┤
│    Provider Packages (@langchain/*)     │  ← OpenAI, Anthropic, Google, etc.
└─────────────────────────────────────────┘
```

## Core Packages

| Package | Install | Purpose |
|---------|---------|---------|
| `langchain` | `npm i langchain` | Agent framework: `createAgent()`, `tool()`, middleware |
| `@langchain/core` | `npm i @langchain/core` | Base abstractions: messages, runnables, tools |
| `@langchain/langgraph` | `npm i @langchain/langgraph` | Graph orchestration: `StateGraph`, checkpointers |
| `deepagents` | `npm i deepagents` | Agent harness: `createDeepAgent()`, filesystem tools |
| `@langchain/openai` | `npm i @langchain/openai` | OpenAI integration |
| `@langchain/anthropic` | `npm i @langchain/anthropic` | Anthropic integration |
| `@langchain/google-genai` | `npm i @langchain/google-genai` | Google Gemini integration |

## Quick Start: Create an Agent

```typescript
import { createAgent, tool } from "langchain";
import * as z from "zod";

const getWeather = tool(
  ({ city }) => `It's always sunny in ${city}!`,
  {
    name: "get_weather",
    description: "Get the weather for a given city",
    schema: z.object({
      city: z.string().describe("The city to get the weather for"),
    }),
  }
);

const agent = createAgent({
  model: "openai:gpt-5.4",
  tools: [getWeather],
});

const result = await agent.invoke({
  messages: [{ role: "user", content: "What's the weather in San Francisco?" }],
});
```

## Core Concepts

### 1. Models

Models are the reasoning engine. Initialize via model identifier strings or directly:

```typescript
// Model identifier string (recommended)
const agent = createAgent({
  model: "openai:gpt-5.4",
  // or "anthropic:claude-sonnet-4-6"
  // or "google-genai:gemini-2.5-flash-lite"
});

// Direct instantiation
import { ChatOpenAI } from "@langchain/openai";
const model = new ChatOpenAI({ model: "gpt-5.4" });

// Dynamic model selection
const agent = createAgent({
  model: (state) => state.messageCount > 10 ? "openai:gpt-5.4" : "openai:gpt-5-nano",
});
```

**Supported capabilities:** Tool calling, structured output, multimodality (images/audio/video), reasoning.

### 2. Tools

Tools extend agent capabilities. Defined with Zod schemas:

```typescript
import { tool } from "langchain";
import * as z from "zod";

// Basic tool
const searchTool = tool(
  ({ query, limit }) => `Found ${limit} results for '${query}'`,
  {
    name: "search_database",
    description: "Search the customer database",
    schema: z.object({
      query: z.string().describe("Search terms"),
      limit: z.number().describe("Max results"),
    }),
  }
);

// Tool with context access
const contextTool = tool(
  async (input, config) => {
    const userId = config.configurable?.userId;
    return `Data for user ${userId}`;
  },
  {
    name: "get_user_data",
    description: "Get user data",
    schema: z.object({ userId: z.string() }),
  }
);

// Dynamic tool registration at runtime
const agent = createAgent({
  model: "openai:gpt-5.4",
  tools: [searchTool],
  middleware: [
    // wrap_model: inject dynamic tools into request
    // wrap_tool_call: handle execution of dynamic tools
  ],
});
```

**Tool naming:** Use `snake_case` (e.g., `web_search`, not `Web Search`). Alphanumeric, underscores, hyphens only.

### 3. Messages

Messages are the fundamental unit of context:

```typescript
import { HumanMessage, SystemMessage, AIMessage, ToolMessage } from "langchain";

// Message types
const system = new SystemMessage("You are a helpful assistant");
const human = new HumanMessage("Hello!");
const ai = new AIMessage("Hi there!");
const toolResult = new ToolMessage({
  tool_call_id: "call_123",
  content: "Tool result here",
});

// Dictionary format (OpenAI-compatible)
const messages = [
  { role: "system", content: "You are helpful" },
  { role: "user", content: "Hello" },
];

// Multimodal content
const imageMessage = new HumanMessage({
  content: [
    { type: "text", text: "Describe this image" },
    { type: "image_url", image_url: { url: "https://example.com/image.jpg" } },
  ],
});
```

**Message roles:** `system`, `user`, `assistant`, `tool`.

### 4. Agents (createAgent)

`createAgent()` builds a graph-based ReAct agent:

```typescript
import { createAgent } from "langchain";

const agent = createAgent({
  // Required
  model: "openai:gpt-5.4",
  tools: [searchTool, calculatorTool],
  
  // Optional configuration
  systemPrompt: "You are a research assistant. Always cite sources.",
  prompt: (state) => `Messages so far: ${state.messages.length}`, // Dynamic prompt
  responseFormat: z.object({ answer: z.string(), confidence: z.number() }), // Structured output
  middleware: [summarizationMiddleware, humanInTheLoopMiddleware],
});

// Invocation
const result = await agent.invoke({
  messages: [{ role: "user", content: "Research quantum computing" }],
});
```

**Agent loop:** ReAct pattern - Reason → Act (tool call) → Observe → Repeat until final answer.

### 5. LangGraph (Orchestration)

For low-level control, build stateful workflows with `StateGraph`:

```typescript
import { StateGraph, START, END } from "@langchain/langgraph";
import { Annotation } from "@langchain/langgraph";

// Define state schema
const StateAnnotation = Annotation.Root({
  messages: Annotation<string[]>({
    reducer: (x, y) => x.concat(y),
    default: () => [],
  }),
  topic: Annotation<string>,
});

// Nodes
function generateTopic(state: typeof StateAnnotation.State) {
  return { topic: "AI agents" };
}

function writeJoke(state: typeof StateAnnotation.State) {
  return { messages: [`Why did the ${state.topic} cross the road?`] };
}

// Build graph
const graph = new StateGraph(StateAnnotation)
  .addNode("generateTopic", generateTopic)
  .addNode("writeJoke", writeJoke)
  .addEdge(START, "generateTopic")
  .addEdge("generateTopic", "writeJoke")
  .addEdge("writeJoke", END)
  .compile();

// Execute
const result = await graph.invoke({ messages: [] });
```

**Key concepts:**
- **State**: Shared data structure representing application snapshot
- **Nodes**: Functions that receive state, perform computation, return updates
- **Edges**: Functions determining next node based on state
- **Super-steps**: Discrete iterations where parallel nodes execute together

### 6. Persistence & Memory

LangGraph provides built-in checkpointing for persistence:

```typescript
import { MemorySaver } from "@langchain/langgraph";

// In-memory (development)
const checkpointer = new MemorySaver();
const graph = builder.compile({ checkpointer });

// Production: Postgres
import { PostgresSaver } from "@langchain/langgraph-checkpoint-postgres";
const pgCheckpointer = PostgresSaver.fromConnString(DB_URI);
await pgCheckpointer.setup(); // Run once

// Production: MongoDB
import { MongoDBSaver } from "@langchain/langgraph-checkpoint-mongodb";
const mongoCheckpointer = new MongoDBSaver({ client });

// Thread-based memory
const config = { configurable: { thread_id: "conversation-123" } };
await graph.invoke({ messages: [msg1] }, config);
await graph.invoke({ messages: [msg2] }, config); // Remembers msg1
```

**Features enabled by persistence:**
- **Short-term memory**: Multi-turn conversations within a thread
- **Long-term memory**: Store user data across sessions via `Store` API
- **Human-in-the-loop**: Pause execution for approval
- **Time travel**: Replay from checkpoints, fork to explore alternatives
- **Fault tolerance**: Resume from last successful step after errors

### 7. Streaming

Stream real-time updates from graph execution:

```typescript
// Stream modes: "values", "updates", "messages", "custom", "tools", "debug"
for await (const chunk of await graph.stream(inputs, { streamMode: "updates" })) {
  console.log(chunk);
}

// Multiple modes
for await (const chunk of await graph.stream(inputs, {
  streamMode: ["messages", "updates"],
})) {
  if (chunk.event === "on_chat_model_stream") {
    process.stdout.write(chunk.data.chunk.content);
  }
}
```

### 8. Interrupts (Human-in-the-Loop)

Pause execution and wait for external input:

```typescript
import { interrupt, Command } from "@langchain/langgraph";

async function approvalNode(state: State) {
  const approved = await interrupt("Do you approve this action?");
  return { approved };
}

// Resume with Command
await graph.invoke(
  new Command({ resume: true }),
  { configurable: { thread_id: "same-thread-id" } }
);
```

### 9. Middleware

Intercept and customize agent execution:

```typescript
import {
  createAgent,
  summarizationMiddleware,
  humanInTheLoopMiddleware,
  modelCallLimitMiddleware,
  toolCallLimitMiddleware,
  modelFallbackMiddleware,
  modelRetryMiddleware,
  toolRetryMiddleware,
  piiMiddleware,
  llmToolSelectorMiddleware,
  contextEditingMiddleware,
} from "langchain";

const agent = createAgent({
  model: "openai:gpt-5.4",
  tools: [...],
  middleware: [
    summarizationMiddleware(),           // Auto-summarize long conversations
    humanInTheLoopMiddleware(),          // Pause for human approval
    modelCallLimitMiddleware({ maxCalls: 10 }), // Limit model calls
    toolCallLimitMiddleware({ maxCalls: 20 }),  // Limit tool calls
    modelFallbackMiddleware({ fallback: "anthropic:claude-3-haiku" }),
    modelRetryMiddleware({ maxRetries: 3 }),
    toolRetryMiddleware({ maxRetries: 3 }),
    piiMiddleware(),                     // Detect/redact PII
    llmToolSelectorMiddleware(),         // Dynamic tool selection
    contextEditingMiddleware(),          // Trim/clear tool uses
  ],
});

// Custom middleware
import { createMiddleware } from "langchain";
const customMiddleware = createMiddleware({
  wrapModel: async (params, next) => {
    console.log("Before model call");
    const result = await next(params);
    console.log("After model call");
    return result;
  },
});
```

**Middleware hooks:** `wrapModel`, `wrapToolCall`, `wrapToolResult`, `transformState`.

### 10. Structured Output

Get typed responses from agents:

```typescript
import { toolStrategy, providerStrategy } from "langchain";

const agent = createAgent({
  model: "openai:gpt-5.4",
  tools: [...],
  responseFormat: z.object({
    answer: z.string(),
    confidence: z.number(),
    sources: z.array(z.string()),
  }),
  // Or enforce strategy:
  // responseFormat: providerStrategy(z.object({...})) // Native structured output
  // responseFormat: toolStrategy(z.object({...}))     // Tool-calling strategy
});

// Result is in structuredResponse key
const result = await agent.invoke({ messages: [...] });
console.log(result.structuredResponse);
```

### 11. Deep Agents

Higher-level agent harness with built-in capabilities:

```typescript
import { createDeepAgent } from "deepagents";

const agent = createDeepAgent({
  tools: [getWeather],
  systemPrompt: "You are a helpful assistant",
  // Built-in capabilities:
  // - Task planning (write_todos tool)
  // - Filesystem tools (ls, read_file, write_file, edit_file)
  // - Subagent spawning (task tool)
  // - Auto-summarization
  // - Memory persistence
  // - Permission rules
  // - Human-in-the-loop
});

const result = await agent.invoke({
  messages: [{ role: "user", content: "Build a React component" }],
});
```

## Integration Providers

### Chat Models

| Provider | Package | Example Model |
|----------|---------|---------------|
| OpenAI | `@langchain/openai` | `openai:gpt-5.4` |
| Anthropic | `@langchain/anthropic` | `anthropic:claude-sonnet-4-6` |
| Google | `@langchain/google-genai` | `google-genai:gemini-2.5-flash` |
| Azure | `@langchain/azure-openai` | `azure-openai:gpt-5.4` |
| AWS Bedrock | `@langchain/aws` | `bedrock:anthropic.claude-3` |
| Cohere | `@langchain/cohere` | `cohere:command-r` |
| Mistral | `@langchain/mistralai` | `mistralai:mistral-large` |
| Ollama | `@langchain/ollama` | `ollama:llama3.2` |
| Groq | `@langchain/groq` | `groq:llama-3.1-70b` |

### Vector Stores

| Provider | Package |
|----------|---------|
| Pinecone | `@langchain/pinecone` |
| Qdrant | `@langchain/qdrant` |
| Weaviate | `@langchain/weaviate` |
| Chroma | `@langchain/chroma` |
| MongoDB | `@langchain/mongodb` |
| PGVector | `@langchain/pgvector` |
| Azure CosmosDB | `@langchain/azure-cosmosdb` |

### Document Loaders

| Source | Package |
|--------|---------|
| PDF | `@langchain/pdf` |
| CSV | `@langchain/csv` |
| Cheerio (HTML) | `@langchain/cheerio` |
| GitHub | `@langchain/github` |
| Notion | `@langchain/notion` |

## Best Practices

1. **Use snake_case for tool names** to ensure cross-provider compatibility
2. **Write clear tool descriptions** - the model uses these to decide when to call tools
3. **Provide context in prompts** - guide agent behavior with system prompts
4. **Handle tool errors gracefully** - tools should catch errors and return informative messages
5. **Use structured output** for predictable, typed responses
6. **Limit iterations** with `toolCallLimitMiddleware` or `modelCallLimitMiddleware` to prevent infinite loops
7. **Use checkpointers in production** - MemorySaver for dev, Postgres/MongoDB for production
8. **Thread IDs for conversations** - Reuse `thread_id` for multi-turn conversations
9. **Model profiles** - LangChain reads provider capabilities dynamically; override with custom `ModelProfile` if needed
10. **Prefer provider strings** - Use `"openai:gpt-5.4"` instead of instantiating `ChatOpenAI` directly for portability

## Common Patterns

### Multi-Agent System

```typescript
const researchAgent = createAgent({ model: "openai:gpt-5.4", tools: [searchTool] });
const writerAgent = createAgent({ model: "openai:gpt-5.4", tools: [writeTool] });

// Use subgraphs in LangGraph
const graph = new StateGraph(State)
  .addNode("research", researchAgent)
  .addNode("write", writerAgent)
  .addEdge(START, "research")
  .addEdge("research", "write")
  .compile();
```

### RAG Pipeline

```typescript
import { RecursiveCharacterTextSplitter } from "@langchain/textsplitters";
import { OpenAIEmbeddings } from "@langchain/openai";
import { MemoryVectorStore } from "langchain/vectorstores/memory";

// 1. Load and split documents
const splitter = new RecursiveCharacterTextSplitter({ chunkSize: 1000 });
const docs = await splitter.splitDocuments(rawDocs);

// 2. Create embeddings and store
const embeddings = new OpenAIEmbeddings();
const vectorStore = await MemoryVectorStore.fromDocuments(docs, embeddings);

// 3. Create retriever tool
const retriever = vectorStore.asRetriever({ k: 4 });
const retrieverTool = createRetrieverTool(retriever, {
  name: "document_search",
  description: "Search through documents",
});

// 4. Create agent with retrieval
const agent = createAgent({
  model: "openai:gpt-5.4",
  tools: [retrieverTool],
});
```

## Resources

- **LangChain JS Docs**: https://docs.langchain.com/oss/javascript/langchain/overview
- **LangGraph Docs**: https://docs.langchain.com/oss/javascript/langgraph/overview
- **Deep Agents Docs**: https://docs.langchain.com/oss/javascript/deepagents/overview
- **API Reference**: https://reference.langchain.com/javascript/
- **GitHub**: https://github.com/langchain-ai/langchainjs

## When to Use What

| Need | Use |
|------|-----|
| Quick agent with tools | `createAgent()` from `langchain` |
| Complex multi-step workflows | `StateGraph` from `@langchain/langgraph` |
| Planning, filesystem, subagents | `createDeepAgent()` from `deepagents` |
| Durable execution, persistence | LangGraph with checkpointers |
| Human approval, time travel | LangGraph interrupts + checkpoints |
| Simple LLM calls without agents | `initChatModel()` or provider classes |
| Tracing/debugging | LangSmith |
