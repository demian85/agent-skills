# grammY Patterns & Best Practices

Project structure, common patterns, and pitfalls to avoid.

## Recommended Project Structure

```
my-telegram-bot/
├── src/
│   ├── bot.ts              # Bot instance, plugins, startup
│   ├── commands/
│   │   ├── start.ts        # /start handler
│   │   ├── help.ts         # /help handler
│   │   └── index.ts        # Export all commands, register with bot
│   ├── keyboards/
│   │   ├── main.ts         # Reply keyboard definitions
│   │   └── inline.ts       # Inline keyboard definitions
│   ├── menus/
│   │   └── settings.ts     # Menu plugin definitions
│   ├── conversations/
│   │   └── signup.ts       # Conversation handlers
│   ├── middleware/
│   │   ├── auth.ts         # Auth/authorization middleware
│   │   ├── logging.ts      # Update logging
│   │   └── error-handler.ts # Error boundary setup
│   ├── sessions/
│   │   └── index.ts        # Session types and storage config
│   ├── types.ts            # Custom context flavors
│   └── config.ts           # Environment/config loading
├── locales/                # i18n translation files
├── .env
├── .env.example
├── package.json
├── tsconfig.json
└── README.md
```

## Modular Command Registration

```typescript
// src/commands/index.ts
import { Bot } from "grammy";
import { startCommand } from "./start";
import { helpCommand } from "./help";
import { MyContext } from "../types";

export function registerCommands(bot: Bot<MyContext>) {
  bot.command("start", startCommand);
  bot.command("help", helpCommand);
  bot.command(["stats", "status"], statsCommand);
}

// src/commands/start.ts
import { MyContext } from "../types";

export async function startCommand(ctx: MyContext) {
  await ctx.reply("Welcome! Use /help to see commands.");
}
```

## Custom Context with Multiple Flavors

```typescript
// src/types.ts
import { Context, SessionFlavor } from "grammy";
import { ConversationFlavor } from "@grammyjs/conversations";
import { HydrateFlavor } from "@grammyjs/hydrate";

interface SessionData {
  step: string;
  data: Record<string, unknown>;
}

interface BotConfig {
  isAdmin: boolean;
  startTime: number;
}

export type MyContext = Context
  & SessionFlavor<SessionData>
  & ConversationFlavor
  & HydrateFlavor
  & { config: BotConfig };
```

## Middleware Order (Critical)

**Correct order matters.** Register in this sequence:

```typescript
// 1. Error boundaries (outermost)
bot.errorBoundary(errorHandler);

// 2. Logging / analytics
bot.use(loggingMiddleware);

// 3. Sequentialize (before session, if using runner)
bot.use(sequentialize(getSessionKey));

// 4. Session (data must be available for everything below)
bot.use(session({ initial: () => ({}) }));

// 5. Conversations (depends on session)
bot.use(conversations());

// 6. Menus (keyboard interaction)
bot.use(menu);

// 7. Custom context injection
bot.use(configMiddleware);

// 8. Commands
registerCommands(bot);

// 9. General handlers
bot.on("message:text", textHandler);
bot.on("callback_query:data", callbackHandler);

// 10. Catch-all
bot.on("message", unknownHandler);
```

## Environment Configuration

```typescript
// src/config.ts
import { config } from "dotenv";
config();

export const BOT_TOKEN = process.env.BOT_TOKEN!;
export const ADMIN_ID = parseInt(process.env.ADMIN_ID || "0");
export const NODE_ENV = process.env.NODE_ENV || "development";

if (!BOT_TOKEN) {
  throw new Error("BOT_TOKEN is required");
}
```

## Auth Middleware Pattern

```typescript
// src/middleware/auth.ts
import { MiddlewareFn } from "grammy";
import { MyContext } from "../types";
import { ADMIN_ID } from "../config";

export const authMiddleware: MiddlewareFn<MyContext> = async (ctx, next) => {
  ctx.config = {
    ...ctx.config,
    isAdmin: ctx.from?.id === ADMIN_ID,
  };
  await next();
};

export const adminOnly: MiddlewareFn<MyContext> = async (ctx, next) => {
  if (!ctx.config.isAdmin) {
    await ctx.reply("⛔ Admin only.");
    return;
  }
  await next();
};

// Usage
bot.use(authMiddleware);
bot.command("admin", adminOnly, (ctx) => {
  ctx.reply("Admin panel");
});
```

## Conversation Patterns

### Wizard-style Form

```typescript
async function signupConversation(conv: MyConversation, ctx: MyContext) {
  await ctx.reply("What's your name?");
  const nameCtx = await conv.waitFor("message:text");
  const name = nameCtx.msg.text;

  await ctx.reply("What's your email?");
  const emailCtx = await conv.waitFor("message:text");
  const email = emailCtx.msg.text;

  // Validate
  if (!email.includes("@")) {
    await ctx.reply("Invalid email. Start over with /signup");
    return;
  }

  // Save to database (idempotent with external())
  await conv.external(async () => {
    await db.users.create({ name, email });
  });

  await ctx.reply(`Thanks, ${name}! Registration complete.`);
}
```

### Conversation with Menu

```typescript
async function settingsConversation(conv: MyConversation, ctx: MyContext) {
  const menu = new Menu("settings-dynamic")
    .text("Toggle Notifications", async (ctx) => {
      const session = await ctx.session;
      session.notifications = !session.notifications;
      await ctx.answerCallbackQuery(
        session.notifications ? "✅ On" : "❌ Off"
      );
    })
    .text("Done", async (ctx) => {
      await ctx.answerCallbackQuery("Settings saved");
      // Signal conversation to end
    });

  await ctx.reply("Settings:", { reply_markup: menu });

  // Wait for user to press "Done" or send /cancel
  await conv.waitFor(["callback_query:data", "message:text"]);
}
```

## Webhook Setup for Custom VPS

```typescript
// src/bot.ts
import express from "express";
import { Bot, webhookCallback } from "grammy";
import { BOT_TOKEN } from "./config";

const bot = new Bot(BOT_TOKEN);

// ... register handlers ...

const app = express();
app.use(express.json());

// Health check
app.get("/health", (req, res) => res.json({ status: "ok" }));

// Webhook endpoint (use bot token as path for security)
app.post(`/${BOT_TOKEN}`, webhookCallback(bot, "express"));

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});

// Set webhook (run once)
// await bot.api.setWebhook(`https://yourdomain.com/${BOT_TOKEN}`);
```

## Scaling Patterns

### Per-Chat Sequential Processing

```typescript
import { run, sequentialize } from "@grammyjs/runner";

bot.use(sequentialize((ctx) => {
  const chat = ctx.chat?.id.toString();
  const user = ctx.from?.id.toString();
  return [chat, user].filter(Boolean) as string[];
}));

const handle = run(bot);
```

### Multi-Instance with Redis Sessions

```typescript
import { RedisAdapter } from "@grammyjs/storage-redis";

bot.use(session({
  storage: new RedisAdapter({
    url: process.env.REDIS_URL,
  }),
}));
```

## Common Pitfalls

| Pitfall | Solution |
|---------|----------|
| Session is `undefined` | Register `session()` middleware **before** handlers that use `ctx.session` |
| Middleware in wrong order | Session → Conversations → Menus → Commands → Handlers |
| In-memory sessions lost | Use external storage adapter (Redis, PostgreSQL, etc.) |
| Token hardcoded | Use `process.env.BOT_TOKEN` |
| Webhook timeouts | Don't run long operations in webhook handlers; use queues |
| Type errors with plugins | Add all plugin flavors to `MyContext` type |
| Duplicate webhook processing | Telegram retries on timeout; make handlers idempotent |
| Menu not responding | Ensure `bot.use(menu)` is registered before command handlers |
| Conversation not starting | `conversations()` must be before `createConversation()` |
| Flood limits | Use `@grammyjs/auto-retry` or `@grammyjs/transformer-throttler` |
| Inline keyboard not working | Ensure `callbackQuery` handlers are registered |
| File upload fails | Use `new InputFile(path)` not raw string paths |
| `ctx.match` is undefined | Only available in `hears()`, `command()`, or `callbackQuery()` |

## TypeScript Strictness

Always use strict mode:

```json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "noUncheckedIndexedAccess": true
  }
}
```

**Why it matters:** grammY's type system relies on strict mode for filter queries and context narrowing to work correctly. Without strict mode, `ctx.msg.text` may not be properly typed even inside `bot.on("message:text")`.

## Testing Patterns

```typescript
// Test a command handler
import { Bot } from "grammy";

const bot = new Bot("test-token");

// Mock context
const createMockCtx = (text: string) => ({
  msg: { text },
  reply: vi.fn(),
});

bot.command("echo", (ctx) => {
  ctx.reply(ctx.msg.text);
});

// Run handler
const ctx = createMockCtx("hello");
bot.handleUpdate({
  update_id: 1,
  message: { message_id: 1, text: "/echo hello", date: 1, chat: { id: 1, type: "private" } },
});
```

## Debugging Commands

Useful for development:

```typescript
// Log all updates
bot.use(async (ctx, next) => {
  console.log("Update:", ctx.update);
  await next();
});

// Log all API calls
bot.api.config.use(async (prev, method, payload, signal) => {
  console.log("API call:", method, payload);
  return await prev(method, payload, signal);
});
```
