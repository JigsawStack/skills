# Web Search — API Reference

## Endpoint

```
POST https://api.interfaze.ai/v1/chat/completions
```

## Authentication

```ts
const interfaze = createOpenAI({
  baseURL: "https://api.interfaze.ai/v1",
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

## Precontext metadata

The raw search results (title, description, content, snippets, URL for each hit) are available on the response's `precontext` array. For structured queries, Interfaze may also follow up with a web extract pass — that result appears as a separate `precontext` entry with `name: "web_extract"`.

```ts
const { object, response } = await generateObject({
  model: interfaze.chat("interfaze-beta"),
  prompt: "...",
  schema: z.object({ ... }),
});

// @ts-expect-error precontext is not typed
const precontext = response.body?.precontext;
// precontext[0].name === "search", precontext[0].result is the list of hits
```

## Coverage

Interfaze's web search covers:

- Research indexes — biology, finance, medicine, law, arxiv, etc.
- Social platforms — LinkedIn, X, personal sites
- Financial data — stocks, forex, crypto, commodities, SEC filings
- General — news, events, products, company information

## Tips

- Be specific in your search prompt. "Find the price of X on Amazon" works better than "look up X."
- Use structured output schemas when you need machine-readable results.
- A web search is automatically triggered when the prompt requires current information — no extra tool config is needed.
- Combine with other skills: search the web, then use `structured-output` to normalize the results.
