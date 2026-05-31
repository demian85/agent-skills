# grammY Core API Reference

Complete reference for the grammY core library classes, types, and middleware system.

## Bot Class

Extends `Composer<Context>`. The central bot instance.

```typescript
import { Bot } from "grammy";
const bot = new Bot("<token>", options?);
```

### Properties

| Property | Type | Description |
|----------|------|-------------|
| `bot.api` | `Api` | Full Telegram Bot API access |
| `bot.token` | `string` | Bot token |
| `bot.errorHandler` | `ErrorHandler` | Global error handler |
| `bot.me` | `UserFromGetMe` | Bot info (available after `init`) |

### Methods

| Method | Description |
|--------|-------------|
| `bot.on(filter, ...middleware)` | Handle updates matching filter query |
| `bot.hears(trigger, ...middleware)` | Match text/caption with string or RegExp |
| `bot.command(command, ...middleware)` | Handle `/command` messages |
| `bot.callbackQuery(trigger, ...middleware)` | Handle inline button clicks |
| `bot.reaction(reaction, ...middleware)` | Handle message reactions |
| `bot.chatType(type, ...middleware)` | Filter by chat type (private, group, etc.) |
| `bot.inlineQuery(trigger, ...middleware)` | Handle inline queries |
| `bot.filter(predicate, ...middleware)` | Custom filter with type guard |
| `bot.drop(predicate, ...middleware)` | Drop updates where predicate is true |
| `bot.use(...middleware)` | Register middleware for all updates |
| `bot.fork(...middleware)` | Run middleware concurrently |
| `bot.lazy(factory)` | Lazy-loaded middleware |
| `bot.route(router, handlers, fallback?)` | Route to different handlers |
| `bot.branch(predicate, trueMw, falseMw)` | Conditional branching |
| `bot.errorBoundary(handler, ...middleware)` | Protect middleware with error boundary |
| `bot.start(options?)` | Start long polling |
| `bot.stop()` | Stop long polling |
| `bot.init(signal?)` | Initialize (fetch bot info) |
| `bot.catch(handler)` | Set error handler |
| `bot.drop(predicate)` | Drop matching updates |

### Start Options

```typescript
bot.start({
  onStart: (botInfo) => console.log(`Started as ${botInfo.username}`),
  timeout: 30_000,     // getUpdates timeout (ms)
  interval: 0,         // Delay between polls (ms)
  signal: abortSignal, // AbortController signal
});
```

## Context Class

Wraps a Telegram update. Passed to every middleware handler.

```typescript
class Context {
  readonly update: Update;
  readonly api: Api;
  readonly me: UserFromGetMe;
  match: string | RegExpMatchArray | undefined;
}
```

### Shortcut Getters

| Getter | Returns | Description |
|--------|---------|-------------|
| `ctx.msg` | `Message` | First available message (message, edited, channel, etc.) |
| `ctx.chat` | `Chat` | Chat from wherever available |
| `ctx.from` | `User` | User from wherever available |
| `ctx.chatId` | `number \| string \| undefined` | Chat ID shortcut |
| `ctx.msgId` | `number \| undefined` | Message ID shortcut |
| `ctx.message` | `Message \| undefined` | `ctx.update.message` |
| `ctx.editedMessage` | `Message \| undefined` | `ctx.update.edited_message` |
| `ctx.callbackQuery` | `CallbackQuery \| undefined` | `ctx.update.callback_query` |
| `ctx.inlineQuery` | `InlineQuery \| undefined` | `ctx.update.inline_query` |
| `ctx.senderChat` | `Chat \| undefined` | Sender chat for channel posts |
| `ctx.businessConnectionId` | `string \| undefined` | Business connection ID |

### Reply Shortcuts

All shortcuts call `ctx.api` methods with `chat_id` pre-filled:

```typescript
ctx.reply(text, other?)                      // sendMessage
ctx.replyWithPhoto(photo, other?)            // sendPhoto
ctx.replyWithAudio(audio, other?)            // sendAudio
ctx.replyWithDocument(doc, other?)           // sendDocument
ctx.replyWithVideo(video, other?)            // sendVideo
ctx.replyWithAnimation(anim, other?)          // sendAnimation
ctx.replyWithVoice(voice, other?)            // sendVoice
ctx.replyWithVideoNote(note, other?)         // sendVideoNote
ctx.replyWithMediaGroup(media, other?)       // sendMediaGroup
ctx.replyWithLocation(lat, lng, other?)      // sendLocation
ctx.replyWithVenue(lat, lng, title, addr)    // sendVenue
ctx.replyWithContact(phone, name, other?)    // sendContact
ctx.replyWithPoll(question, options, other?) // sendPoll
ctx.replyWithDice(emoji?)                    // sendDice
ctx.replyWithChatAction(action)              // sendChatAction
ctx.replyWithSticker(sticker, other?)        // sendSticker
```

### Entity Extraction

```typescript
ctx.entities()                    // All entities from text/caption
ctx.entities("url")                // Only URL entities
ctx.entities(["url", "email"])     // Multiple types
```

### Reaction Analysis

```typescript
ctx.reactions()
// Returns: { emoji, emojiAdded, emojiKept, emojiRemoved, customEmoji, ... }
```

### Type Guards

```typescript
ctx.has(":text")              // Has text content
ctx.hasCommand("start")       // Has /start command
ctx.hasCallbackQuery("data")  // Has callback query with data
```

## Composer Class

Superclass of `Bot`. Heart of the middleware system.

```typescript
const composer = new Composer<Context>(...middleware);
```

### Middleware Registration

| Method | Purpose |
|--------|---------|
| `composer.use(...mw)` | Register for all updates |
| `composer.on(filter, ...mw)` | Filter by update type |
| `composer.hears(trigger, ...mw)` | Match text content |
| `composer.command(trigger, ...mw)` | Match bot commands |
| `composer.callbackQuery(trigger, ...mw)` | Match callback queries |
| `composer.inlineQuery(trigger, ...mw)` | Match inline queries |
| `composer.filter(predicate, ...mw)` | Custom filter function |
| `composer.drop(predicate, ...mw)` | Drop matching updates |
| `composer.fork(...mw)` | Run concurrently |
| `composer.lazy(factory)` | Lazy-loaded middleware |
| `composer.route(router, handlers, fallback?)` | Route to handlers |
| `composer.branch(pred, trueMw, falseMw)` | Conditional |
| `composer.errorBoundary(handler, ...mw)` | Error boundary |

### Chaining

```typescript
bot.on("message")
  .filter(ctx => ctx.from?.id === 123456789)
  .hears("hello", ctx => ctx.reply("Hi owner!"));
```

## Api Class

Low-level Telegram Bot API access.

```typescript
const api = new Api(token, options?);
```

### Properties

| Property | Description |
|----------|-------------|
| `api.raw` | Raw API (1:1 with Telegram HTTP API) |
| `api.config` | `{ use: TransformerConsumer }` — Install transformers |
| `api.token` | Bot token |

### Common Methods

```typescript
api.sendMessage(chat_id, text, other?)
api.sendPhoto(chat_id, photo, other?)
api.getFile(file_id)
api.getMe()
api.setWebhook(url, other?)
api.deleteWebhook(other?)
api.getWebhookInfo()
// ... 150+ methods matching Telegram Bot API
```

## TypeScript Types

### FilterQuery

Specifies which updates to handle:

```typescript
type FilterQuery =
  | "message" | "edited_message" | "channel_post" | "edited_channel_post"
  | "callback_query" | "inline_query" | "chosen_inline_result"
  | "shipping_query" | "pre_checkout_query" | "poll" | "poll_answer"
  | "my_chat_member" | "chat_member" | "chat_join_request"
  | "chat_boost" | "removed_chat_boost" | "message_reaction"
  | "message_reaction_count" | "business_connection"
  // + nested paths like "message:text", "callback_query:data"
```

### Context Flavors

```typescript
// Session flavor
interface SessionFlavor<S> {
  session: S;
}
type MyContext = Context & SessionFlavor<SessionData>;

// Lazy session flavor
interface LazySessionFlavor<S> {
  session: Promise<S>;
}
type MyContext = Context & LazySessionFlavor<SessionData>;

// Conversation flavor
interface ConversationFlavor {
  conversation: ConversationControls;
}

// File flavor (from @grammyjs/files)
interface FileFlavor {
  getFile(): Promise<File>;
}
```

### Middleware Types

```typescript
type MiddlewareFn<C extends Context = Context> =
  (ctx: C, next: NextFunction) => MaybePromise<unknown>;

interface MiddlewareObj<C extends Context = Context> {
  middleware: () => MiddlewareFn<C>;
}

type Middleware<C extends Context = Context> =
  | MiddlewareFn<C>
  | MiddlewareObj<C>;

type NextFunction = () => Promise<void>;
```

### Error Types

```typescript
// API error (Telegram returned ok: false)
class GrammyError extends Error {
  readonly ok: false;
  readonly error_code: number;
  readonly description: string;
  readonly parameters: ResponseParameters;
  readonly method: string;
  readonly payload: Record<string, unknown>;
}

// Network/HTTP failure
class HttpError extends Error {
  readonly error: unknown;
}

// Middleware error wrapper
class BotError<C extends Context> extends Error {
  readonly error: unknown;
  readonly ctx: C;
}
```

### Configuration Types

```typescript
interface BotConfig<C extends Context = Context> {
  client?: ApiClientOptions;
  botInfo?: UserFromGetMe;      // Skip getMe call
  ContextConstructor?: new (...args) => C;
}

interface ApiClientOptions {
  apiRoot?: string;             // Default: "https://api.telegram.org"
  tokenBot?: string;            // Default: "bot"
  timeout?: number;             // Request timeout (ms)
  signal?: AbortSignal;
  canRetry?: boolean;
  maxRetries?: number;
  retryAfter?: number;          // ms between retries
  agent?: Agent;                // Node.js HTTP agent
}

interface PollingOptions {
  interval?: number;            // Delay between polls (ms)
  timeout?: number;             // getUpdates timeout (ms)
  signal?: AbortSignal;
  onStart?: (botInfo: UserFromGetMe) => void;
}
```

## Middleware Execution Order

```typescript
// First registered = first executed
bot.use(mwA);              // 1. Runs first
bot.use(mwB);              // 2. Runs second
bot.on("message", mwC);    // 3. Runs third (only for messages)

// Before/after next()
bot.use(async (ctx, next) => {
  console.log("A: before");
  await next();
  console.log("A: after");   // Runs after all downstream middleware
});
```

## Common Imports

```typescript
// Core
import { Bot, Context, Composer, Api, GrammyError, HttpError, BotError } from "grammy";

// Sessions
import { session, lazySession, SessionFlavor, LazySessionFlavor, MemorySessionStorage } from "grammy";

// Input files
import { InputFile } from "grammy";

// Keyboard builders
import { Keyboard, InlineKeyboard } from "grammy";

// Filter helpers
import { matchFilter, Filter } from "grammy";

// Webhook
import { webhookCallback } from "grammy";
```
