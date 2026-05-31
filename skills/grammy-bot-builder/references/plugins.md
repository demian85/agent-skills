# grammY Plugins Reference

Official and built-in plugins for grammY. All support TypeScript.

## Built-in Plugins (No install needed)

### Sessions

Store per-chat or per-user data.

```typescript
import { Bot, Context, session, SessionFlavor, MemorySessionStorage } from "grammy";

interface SessionData {
  count: number;
}

type MyContext = Context & SessionFlavor<SessionData>;
const bot = new Bot<MyContext>("");

bot.use(session({
  initial: (): SessionData => ({ count: 0 }),
  storage: new MemorySessionStorage(), // Default, lost on restart
}));
```

**Session options:**

| Option | Default | Description |
|--------|---------|-------------|
| `initial` | required | Factory function returning default session data |
| `storage` | `MemorySessionStorage` | Storage adapter |
| `getSessionKey` | `(ctx) => ctx.chat?.id.toString()` | Key function for storage |

**Lazy sessions:**
```typescript
import { lazySession, LazySessionFlavor } from "grammy";
type MyContext = Context & LazySessionFlavor<SessionData>;
bot.use(lazySession({ initial: () => ({ count: 0 }) }));
// Access: const session = await ctx.session;
```

**Multi-session:**
```typescript
bot.use(session({
  type: "multi",
  user: { storage: redisAdapter, initial: () => ({}) },
  chat: { storage: psqlAdapter, initial: () => ({}) },
}));
// ctx.session.user and ctx.session.chat from different stores
```

### Keyboards

```typescript
import { Keyboard, InlineKeyboard } from "grammy";

// Reply keyboard
const replyKeyboard = new Keyboard()
  .text("A").text("B")
  .row()
  .text("C")
  .resized()              // Fit keyboard to buttons
  .oneTime()              // Hide after use
  .placeholder("Choose..."); // Input placeholder

// Inline keyboard
const inlineKeyboard = new InlineKeyboard()
  .text("Yes", "callback_yes")
  .text("No", "callback_no")
  .row()
  .url("Open", "https://example.com")
  .switchInlineQuery("share")
  .switchInlineQueryCurrentChat("search")
  .callbackGame("Play");
```

### Media Group

```typescript
import { MediaGroup } from "grammy";

const group = new MediaGroup()
  .photo("photo_id")
  .video("video_id");

ctx.replyWithMediaGroup(group.build());
```

### Inline Query Results

```typescript
import { InlineQueryResultBuilder } from "grammy";

bot.inlineQuery("query", async (ctx) => {
  const results = [
    InlineQueryResultBuilder.article("id1", "Title", {
      input_message_content: { message_text: "Hello!" },
    }),
  ];
  await ctx.answerInlineQuery(results);
});
```

## Official Plugins (npm install required)

### Conversations

Multi-step dialogs with state persistence.

```bash
npm install @grammyjs/conversations
```

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
  await ctx.reply("Name?");
  const { message } = await conv.waitFor("message:text");
  await ctx.reply(`Hello ${message.text}!`);
}

bot.use(createConversation(signup));
bot.command("signup", (ctx) => ctx.conversation.enter("signup"));
```

**Conversation methods:**

| Method | Description |
|--------|-------------|
| `conv.wait()` | Wait for any update |
| `conv.waitFor(filter)` | Wait for specific update type |
| `conv.external(promise)` | Run side effect only once (idempotent) |
| `conv.sleep(ms)` | Pause for milliseconds |
| `conv.skip()` | Skip current update, wait for next |
| `conv.now()` | Current timestamp |

### Menu

Dynamic interactive menus with navigation.

```bash
npm install @grammyjs/menu
```

```typescript
import { Menu } from "@grammyjs/menu";

const mainMenu = new Menu("main")
  .text("Profile", (ctx) => ctx.reply("Profile"))
  .submenu("Settings", "settings")
  .row()
  .text("Back", (ctx) => ctx.menu.back());

const settingsMenu = new Menu("settings")
  .text("Language", (ctx) => ctx.reply("English"))
  .back("Back");

mainMenu.register(settingsMenu);
bot.use(mainMenu);
```

**Menu methods:**

| Method | Description |
|--------|-------------|
| `.text(label, handler)` | Text button |
| `.url(label, url)` | URL button |
| `.webApp(label, url)` | Web App button |
| `.switchInline(label, query)` | Switch inline query |
| `.submenu(label, menuId)` | Navigate to submenu |
| `.back(label)` | Go back to parent menu |
| `.row()` | Start new row |
| `.register(menu)` | Register submenu |

### Runner

Concurrent long polling for high-load bots.

```bash
npm install @grammyjs/runner
```

```typescript
import { run, sequentialize } from "@grammyjs/runner";

// Prevent concurrent access to same chat session
bot.use(sequentialize((ctx) => [ctx.chat?.id.toString()]));

const handle = run(bot); // Concurrent by default

// Graceful shutdown
process.once("SIGINT", () => handle.stop());
```

### Hydrate

Call methods on objects returned from API calls.

```bash
npm install @grammyjs/hydrate
```

```typescript
import { hydrate, HydrateFlavor } from "@grammyjs/hydrate";

type MyContext = HydrateFlavor<Context>;
bot.use(hydrate());

// Without hydrate:
const status = await ctx.reply("Processing");
await ctx.api.editMessageText(ctx.chat.id, status.message_id, "Done!");

// With hydrate:
const status = await ctx.reply("Processing");
await status.editText("Done!"); // Method on the message object
```

### Auto-Retry

Automatically handle rate limits and server errors.

```bash
npm install @grammyjs/auto-retry
```

```typescript
import { autoRetry } from "@grammyjs/auto-retry";

bot.api.config.use(autoRetry({
  maxRetryAttempts: 3,
  maxDelaySeconds: 60,
}));
```

### Commands

Manage bot commands and scopes.

```bash
npm install @grammyjs/commands
```

```typescript
import { commands } from "@grammyjs/commands";

bot.use(commands());

// Set commands with scopes
bot.command("start", (ctx) => ctx.reply("Welcome!"));
```

### Files

Enhanced file handling.

```bash
npm install @grammyjs/files
```

```typescript
import { FileFlavor } from "@grammyjs/files";

type MyContext = Context & FileFlavor;

bot.on(":file", async (ctx) => {
  const file = await ctx.getFile();
  await file.download();  // Download to temp location
  await file.getUrl();    // Get temporary URL
});
```

### Internationalization (i18n)

Multi-language support.

```bash
npm install @grammyjs/i18n
```

```typescript
import { I18n } from "@grammyjs/i18n";

const i18n = new I18n({
  defaultLocale: "en",
  directory: "locales",
});

bot.use(i18n);
bot.command("start", (ctx) => ctx.reply(ctx.t("welcome")));
```

### Router

Route updates to different handlers.

```bash
npm install @grammyjs/router
```

```typescript
import { Router } from "@grammyjs/router";

const router = new Router((ctx) => ctx.session?.route ?? "default");

router.route("start", startHandler);
router.route("settings", settingsHandler);
router.otherwise(defaultHandler);

bot.use(router);
```

### Rate Limiter

Restrict users who spam.

```bash
npm install @grammyjs/ratelimiter
```

```typescript
import { limit } from "@grammyjs/ratelimiter";

bot.use(limit());
```

### Throttler

Slow down API calls to avoid flood limits.

```bash
npm install @grammyjs/transformer-throttler
```

```typescript
import { apiThrottler } from "@grammyjs/transformer-throttler";

bot.api.config.use(apiThrottler());
```

### Parse Mode

Simplify message formatting (HTML/Markdown).

```bash
npm install @grammyjs/parse-mode
```

```typescript
import { parseMode } from "@grammyjs/parse-mode";

bot.use(parseMode("HTML"));
ctx.reply("<b>Bold</b> text"); // Auto-sets parse_mode
```

### Emoji

Simplify emoji usage in code.

```bash
npm install @grammyjs/emoji
```

```typescript
import { emoji } from "@grammyjs/emoji";

ctx.reply(`${emoji("pizza")} Order received!`);
```

### Stateless Question

Create dialogs without data storage.

```bash
npm install @grammyjs/stateless-question
```

```typescript
import { StatelessQuestion } from "@grammyjs/stateless-question";

const nameQuestion = new StatelessQuestion("name", (ctx) => {
  ctx.reply(`Hello, ${ctx.message?.text}!`);
});

bot.use(nameQuestion.middleware());
bot.command("ask", (ctx) => nameQuestion.replyWithMarkdown(ctx, "What's your name?"));
```

## Storage Adapters

Install adapters for persistent session storage:

| Storage | Package | Example |
|---------|---------|---------|
| **File** | `@grammyjs/storage-file` | `new FileAdapter({ dirName: "sessions" })` |
| **Redis** | `@grammyjs/storage-redis` | `new RedisAdapter({ url: "redis://localhost" })` |
| **MongoDB** | `@grammyjs/storage-mongodb` | `new MongoDBAdapter({ collection })` |
| **PostgreSQL** | `@grammyjs/storage-psql` | `new PsqlAdapter({ pool })` |
| **Supabase** | `@grammyjs/storage-supabase` | `new SupabaseAdapter({ supabase })` |
| **Prisma** | `@grammyjs/storage-prisma` | `new PrismaAdapter({ prisma })` |
| **Cloudflare KV** | `@grammyjs/storage-cloudflare` | `new CloudflareAdapter({ namespace })` |
| **DynamoDB** | `@grammyjs/storage-dynamodb` | `new DynamoDBAdapter({ client })` |
| **Deno KV** | `@grammyjs/storage-denokv` | `new DenoKVAdapter({ kv })` |
| **Free** | `@grammyjs/storage-free` | `freeStorage(bot.token)` (cloud, hobby use) |

**Usage:**
```typescript
import { RedisAdapter } from "@grammyjs/storage-redis";

bot.use(session({
  initial: () => ({ count: 0 }),
  storage: new RedisAdapter({ url: "redis://localhost:6379" }),
}));
```

**Enhance storage with TTL:**
```typescript
import { enhanceStorage } from "grammy";

bot.use(session({
  storage: enhanceStorage({
    storage: redisAdapter,
    millisecondsToLive: 30 * 60 * 1000, // 30 min
  }),
}));
```

## Plugin Type Reference

### Middleware Plugins

Return a middleware function, installed via `bot.use(plugin)`:

- `session`
- `conversations`
- `menu`
- `hydrate`
- `files`
- `i18n`
- `router`
- `ratelimiter`
- `commands`

### Transformer Plugins

Return a transformer function, installed via `bot.api.config.use(plugin)`:

- `autoRetry`
- `apiThrottler` (from transformer-throttler)
- `parseMode`

## Common Context Flavor Combinations

```typescript
import { Context, SessionFlavor } from "grammy";
import { ConversationFlavor } from "@grammyjs/conversations";
import { HydrateFlavor } from "@grammyjs/hydrate";
import { FileFlavor } from "@grammyjs/files";
import { I18nFlavor } from "@grammyjs/i18n";

interface SessionData {
  count: number;
}

type MyContext = Context
  & SessionFlavor<SessionData>
  & ConversationFlavor
  & HydrateFlavor
  & FileFlavor
  & I18nFlavor;
```
