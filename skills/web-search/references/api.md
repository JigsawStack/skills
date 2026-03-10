# Web Search — API Reference

## Endpoint

```
POST https://interfaze.ai/v1/chat/completions
```

## Authentication

```ts
const interfaze = createOpenAI({
  baseURL: "https://interfaze.ai/v1",
  apiKey: process.env.INTERFAZE_API_KEY,
});
```

## How it works

Interfaze AI has built-in web search. You do not need to configure a separate search tool or API. Simply include a prompt that requires current information, and the model will search the web as needed.

## Structured search results

Use `generateObject` to get structured results:

```ts
const response = await generateObject({
  model: interfaze.chat("interfaze-beta"),
  prompt: "Search for ...",
  schema: z.object({
    // Define your expected result shape
  }),
});
```

## Plain text search

Use `generateText` for a natural language summary:

```ts
const response = await generateText({
  model: interfaze.chat("interfaze-beta"),
  prompt: "What is the current ...",
});
```

## Tips

- Be specific in your search prompt. "Find the price of X on Amazon" works better than "look up X."
- Use structured output schemas when you need machine-readable results.
- Combine with other skills: search the web, then use `structured-output` to normalize the results.
