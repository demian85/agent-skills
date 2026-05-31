---
name: grammy-bot-builder
description: >
  Comprehensive skill for building Telegram bots with grammY (TypeScript/Node.js/Bun/Deno).
  Use whenever the user mentions Telegram bots, grammY, bot commands, inline keyboards,
  bot webhooks, bot sessions, bot conversations, inline queries, or building a chatbot with
  TypeScript/Node.js/Bun. Also use for debugging bot issues, adding features to existing bots,
  looking up grammY API patterns, or setting up long-polling/webhook bots on a custom VPS.
  Covers core API, all official plugins, middleware patterns, sessions, interactive menus,
  conversations, file handling, error handling, and local hosting.
---

# grammY Bot Builder

Build Telegram bots with [grammY](https://grammy.dev), a modern, type-safe framework for Node.js, Bun, and Deno.

## When to Use This Skill

- Creating a new Telegram bot from scratch
- Adding features to an existing grammY bot
- Debugging bot errors (GrammyError, HttpError, middleware issues)
- Setting up sessions, keyboards, menus, or conversations
- Configuring webhooks or long polling for custom/local hosting
- Looking up grammY API patterns, types, or plugin usage

## Quick Start

### Install

```bash
# Node.js
npm install grammy

# Bun
bun add grammy

# Deno
import { Bot } from "https://deno.land/x/grammy/mod.ts";
```

### Minimal Bot

```typescript
import { Bot } from "grammy";

const bot = new Bot(process.env.BOT_TOKEN!);

bot.command("start", (ctx) => ctx.reply("Hello!"));
bot.on("message:text", (ctx) => ctx.reply(`You said: ${ctx.msg.text}`));

bot.start();
```

### TypeScript Config

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true
  }
}
```

## Core Concepts

### 1. Bot Instance

```typescript
const bot = new Bot("<token>", {
  client: {
    canRetry: true,
    maxRetries: 3,
    timeout: 30_000,
  }
});
```

**Key properties:**
- `bot.api` — Telegram Bot API methods
- `bot.token` — Bot token
- `bot.errorHandler` — Global error handler

**Lifecycle:**
- `await bot.init()` — Fetch bot info (auto-called by `start`)
- `bot.start(options?)` — Begin long polling
- `bot.stop()` — Stop polling
- `bot.catch(handler)` — Set error handler

### 2. Context (`ctx`)

The context wraps every incoming update and provides shortcuts:

```typescript
// Access data
ctx.msg          // Message object
ctx.chat         // Chat object
ctx.from         // User object
ctx.chatId       // Chat ID shortcut
ctx.msgId        // Message ID shortcut

// Reply shortcuts (auto-fill chat_id)
ctx.reply("text")
ctx.replyWithPhoto(fileId)
ctx.replyWithDocument(file)
ctx.replyWithChatAction("typing")

// Entity extraction
ctx.entities()              // All entities
ctx.entities("url")          // URL entities only
ctx.entities(["url", "email"]) // Multiple types
```

### 3. Filter Queries

Use `bot.on()` to handle specific update types:

```typescript
bot.on("message")                    // Any message
bot.on("message:text")               // Text messages
bot.on("message:photo")              // Photos
bot.on("callback_query:data")       // Inline button clicks
bot.on("inline_query")               // Inline queries
bot.on("edited_message")             // Edited messages
bot.on("my_chat_member")             // Bot added/removed/blocked

// Shorthand with omitted prefixes
bot.on(":text")                     // Text in messages or channel posts
bot.on(":media")                    // Photos, videos, etc.
bot.on(":file")                     // Any file type

// Multiple filters (OR)
bot.on(["message", "edited_message"])
```

### 4. Middleware System

Middleware is a chain. **Order matters** — first registered runs first.

```typescript
// Basic middleware
bot.use(async (ctx, next) => {
  console.log("Before");
  await next(); // Continue to next handler
  console.log("After");
});

// Conditional branching
bot.branch(
  (ctx) => ctx.from?.id === OWNER_ID,
  ownerHandler,
  userHandler
);

// Route by value
bot.route(
  (ctx) => ctx.session?.step ?? "default",
  {
    name: nameHandler,
    email: emailHandler,
    default: defaultHandler,
  }
);

// Error boundary
const safe = bot.errorBoundary((err, next) => {
  console.error("Caught:", err);
  // Error does not bubble up past this boundary
});
safe.on("message", riskyHandler);
```

## Project Scaffolding

Recommended structure for a bot project:

```
my-bot/
├── src/
│   ├── bot.ts           # Bot instance, middleware, startup
│   ├── commands.ts      # Command handlers
│   ├── keyboards.ts      # Inline/reply keyboard definitions
│   ├── menus.ts          # Interactive menu definitions
│   ├── conversations.ts  # Conversation handlers
│   ├── sessions.ts       # Session types and config
│   └── types.ts          # Custom context flavors
├── .env                  # BOT_TOKEN=...
├── package.json
└── tsconfig.json
```

See [references/patterns.md](references/patterns.md) for full project patterns, modular routing, and scaling strategies.

## Sessions & State

Sessions store per-chat or per-user data:

```typescript
import { Bot, Context, session, SessionFlavor } from "grammy";

interface SessionData {
  count: number;
}

type MyContext = Context & SessionFlavor<SessionData>;

const bot = new Bot<MyContext>("");

bot.use(session({
  initial: (): SessionData => ({ count: 0 }),
}));

bot.on("message", (ctx) => {
  ctx.session.count++;
  ctx.reply(`Count: ${ctx.session.count}`);
});
```

**Session keys** (default = per chat):
```typescript
bot.use(session({
  getSessionKey: (ctx) => ctx.from?.id.toString(), // Per user
}));
```

**Storage adapters:** Memory (default), File, Redis, MongoDB, PostgreSQL, Supabase, Prisma, and more. See [references/plugins.md](references/plugins.md).

**Lazy sessions** (performance optimization):
```typescript
import { lazySession, LazySessionFlavor } from "grammy";

type MyContext = Context & LazySessionFlavor<SessionData>;
bot.use(lazySession({ initial: () => ({ count: 0 }) }));
// Access with await: const session = await ctx.session;
```

## Interactive Features

### Reply Keyboards

```typescript
import { Keyboard } from "grammy";

const keyboard = new Keyboard()
  .text("Option A").text("Option B")
  .row()
  .text("Cancel")
  .resized();

ctx.reply("Choose:", { reply_markup: keyboard });
```

### Inline Keyboards

```typescript
import { InlineKeyboard } from "grammy";

const inline = new InlineKeyboard()
  .text("Yes", "yes")
  .text("No", "no")
  .row()
  .url("Open site", "https://example.com");

ctx.reply("Confirm?", { reply_markup: inline });

// Handle callbacks
bot.callbackQuery("yes", (ctx) => {
  ctx.answerCallbackQuery("Confirmed!");
});
```

### Interactive Menus (Plugin)

```typescript
import { Menu } from "@grammyjs/menu";

const mainMenu = new Menu("main")
  .text("Profile", (ctx) => ctx.reply("Your profile"))
  .submenu("Settings", "settings");

const settings = new Menu("settings")
  .text("Language", (ctx) => ctx.reply("English"))
  .back("Back");

mainMenu.register(settings);
bot.use(mainMenu);

bot.command("menu", (ctx) =>
  ctx.reply("Menu:", { reply_markup: mainMenu })
);
```

### Conversations (Plugin)

Multi-step dialogs that survive restarts:

```typescript
import {
  conversations,
  createConversation,
  type Conversation,
  type ConversationFlavor,
} from "@grammyjs/conversations";

type MyContext = Context & ConversationFlavor;
type MyConversation = Conversation<MyContext>;

bot.use(conversations());

async function signup(conv: MyConversation, ctx: MyContext) {
  await ctx.reply("What's your name?");
  const { message } = await conv.waitFor("message:text");
  await ctx.reply(`Hello, ${message.text}!`);
}

bot.use(createConversation(signup));
bot.command("signup", (ctx) => ctx.conversation.enter("signup"));
```

## Error Handling

### Error Types

```typescript
import { GrammyError, HttpError, BotError } from "grammy";

bot.catch((err) => {
  if (err.error instanceof GrammyError) {
    // API error (e.g., chat not found, message too long)
    console.error("API Error:", err.error.description);
  } else if (err.error instanceof HttpError) {
    // Network failure
    console.error("Network Error:", err.error.error);
  } else {
    // Unknown error
    console.error("Unexpected Error:", err.error);
  }
});
```

### Error Boundaries

```typescript
const safeZone = bot.errorBoundary((err, next) => {
  console.error("Protected area error:", err);
  // Optionally continue: await next();
});

safeZone.on("message", handlerWithPossibleBugs);
```

## File Handling

### Receiving Files

```typescript
bot.on("message:document", async (ctx) => {
  const doc = ctx.msg.document;
  const file = await ctx.getFile();
  const url = `https://api.telegram.org/file/bot${bot.token}/${file.file_path}`;
  // Download from URL (valid for 1 hour)
});
```

### Sending Files

```typescript
import { InputFile } from "grammy";

// By file_id (fastest, reuses Telegram storage)
await ctx.replyWithPhoto(existingFileId);

// By URL (Telegram downloads)
await ctx.replyWithPhoto("https://example.com/image.jpg");

// Upload local file
await ctx.replyWithPhoto(new InputFile("/path/to/image.jpg"));

// From buffer/stream
await ctx.replyWithDocument(new InputFile(buffer));
await ctx.replyWithDocument(new InputFile(createReadStream("file.pdf")));
```

## Scaling & Local Hosting

### Long Polling (Default)

```typescript
// Simple sequential polling
bot.start();

// Production: grammY runner for concurrency
import { run, sequentialize } from "@grammyjs/runner";

bot.use(sequentialize((ctx) => [ctx.chat?.id.toString()]));
const handle = run(bot);

// Graceful shutdown
process.once("SIGINT", () => handle.stop());
process.once("SIGTERM", () => handle.stop());
```

### Webhooks (Custom VPS)

```typescript
import express from "express";
import { webhookCallback } from "grammy";

const app = express();
app.use(express.json());

// Mount webhook handler
app.post(`/${bot.token}`, webhookCallback(bot, "express"));

app.listen(3000, () => {
  console.log("Webhook server running on port 3000");
});

// Set webhook URL via Telegram API
// https://api.telegram.org/bot<token>/setWebhook?url=<url>/<token>
```

**Supported adapters:** express, fastify, koa, hono, next-js, aws-lambda, cloudflare.

**⚠️ Warning:** Don't run long operations in webhook handlers — Telegram times out after ~10s and may resend the update.

## Custom Context Flavors

Extend context with your own properties:

```typescript
interface BotConfig {
  isAdmin: boolean;
}

type MyContext = Context & { config: BotConfig };

const bot = new Bot<MyContext>("");

bot.use(async (ctx, next) => {
  ctx.config = { isAdmin: ctx.from?.id === ADMIN_ID };
  await next();
});

bot.command("admin", (ctx) => {
  if (ctx.config.isAdmin) ctx.reply("Welcome, admin!");
});
```

## Key Plugins Quick Reference

| Plugin | Install | Purpose |
|--------|---------|---------|
| sessions | built-in | Per-user/chat state storage |
| keyboards | built-in | Inline and reply keyboard builders |
| conversations | `npm i @grammyjs/conversations` | Multi-step dialogs |
| menu | `npm i @grammyjs/menu` | Interactive button menus |
| runner | `npm i @grammyjs/runner` | Concurrent long polling |
| hydrate | `npm i @grammyjs/hydrate` | Call methods on API objects |
| auto-retry | `npm i @grammyjs/auto-retry` | Automatic rate limit handling |
| commands | `npm i @grammyjs/commands` | Command management |
| files | `npm i @grammyjs/files` | File handling helpers |
| i18n | `npm i @grammyjs/i18n` | Internationalization |

See [references/plugins.md](references/plugins.md) for full plugin details.

## Debugging Checklist

When a bot isn't working:

1. **Token valid?** Check with `curl https://api.telegram.org/bot<token>/getMe`
2. **Middleware order?** Session → Conversations → Menus → Commands → Handlers
3. **Session undefined?** Ensure `session()` middleware is registered before handlers that use `ctx.session`
4. **Webhook not firing?** Verify webhook URL is set and reachable from the internet
5. **Duplicate messages?** Don't use `return next()` in webhook handlers without proper timeout handling
6. **Type errors with plugins?** Add the plugin's flavor to your context type: `type MyContext = Context & SessionFlavor & ConversationFlavor`
7. **Files not sending?** Use `new InputFile()` for local files, not raw paths
8. **Menu buttons not working?** Ensure `bot.use(menu)` is called before command handlers
9. **Conversation not starting?** Check `bot.use(conversations())` is before `createConversation()`
10. **Rate limited?** Install `@grammyjs/auto-retry` or use `@grammyjs/runner` with `sequentialize`

## References

- [references/core-api.md](references/core-api.md) — Complete API reference for Bot, Context, Composer, types, and middleware
- [references/plugins.md](references/plugins.md) — All official plugins with install commands and usage examples
- [references/patterns.md](references/patterns.md) — Project structure, common patterns, modular routing, and pitfalls
- [grammY Docs](https://grammy.dev)
- [Telegram Bot API](https://core.telegram.org/bots/api)
