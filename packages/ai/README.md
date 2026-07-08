# @opencoredev/loginwithchatgpt-ai

Vercel AI SDK provider for [Login with ChatGPT](../../README.md).

Use `createChatGPTProxyProvider()` in browsers and app-server routes.
`createChatGPT()` is for headless/server-only flows where you intentionally
manage token custody yourself.

## Browser proxy mode

```ts
import { createChatGPTProxyProvider } from "@opencoredev/loginwithchatgpt-ai";
import { streamText } from "ai";

const chatgpt = createChatGPTProxyProvider({ basePath: "/api/chatgpt" });
const models = await chatgpt.listModels();
const model = models.includes("gpt-5.5")
  ? "gpt-5.5"
  : models[0];

if (!model) throw new Error("No ChatGPT models were returned for this account.");

const result = streamText({
  model: chatgpt(model),
  prompt,
});
```

The proxy provider sends requests to your server handler. In browser code it
uses the session cookie; in server routes, pass `auth.proxyFetch(request)` so
tokens stay inside the handler:

```ts
const chatgpt = createChatGPTProxyProvider({
  fetch: auth.proxyFetch(request),
});
```

## Direct token mode

```ts
import { createChatGPT } from "@opencoredev/loginwithchatgpt-ai";
import { streamText } from "ai";

const chatgpt = createChatGPT({
  credentials: tokens,
  onRefresh: saveTokens,
});

const models = await chatgpt.listModels();
const model = models.includes("gpt-5.5")
  ? "gpt-5.5"
  : models[0];

if (!model) throw new Error("No ChatGPT models were returned for this account.");

const result = streamText({
  model: chatgpt(model),
  prompt,
});
```

Direct token mode is an escape hatch for CLIs, headless apps, or migrations.
Normal web apps should keep using the proxy provider.

To request Codex Fast tier through the browser proxy, pass the SDK header:

```ts
const result = streamText({
  model: chatgpt("gpt-5.5"),
  prompt,
  headers: {
    "x-login-with-chatgpt-service-tier": "fast",
  },
});
```

## Provider shape

```ts
chatgpt(modelId);
chatgpt.responses(modelId);
await chatgpt.listModels();
chatgpt.openai;
```

Peer dependencies: `ai@^7` and `@ai-sdk/openai@^4`.
