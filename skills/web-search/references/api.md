# Web Search — API Reference

## Endpoint

```
POST https://api.interfaze.ai/v1/chat/completions
```

Model name: `interfaze-beta`.

## How it works

Interfaze AI has built-in web search. You do not need to configure a separate search tool or API. Send a prompt that requires current information and the model searches the web as needed.

## Structured search results

Use the structured-output mechanism of your SDK to get typed results:

```ts
// TypeScript — Vercel AI SDK
const { object } = await generateObject({
  model: interfaze.chat("interfaze-beta"),
  schema: z.object({ /* expected result shape */ }),
  prompt: "Search for ...",
});
```

```python
# Python — OpenAI SDK
response = interfaze.chat.completions.create(
    model="interfaze-beta",
    messages=[{"role": "user", "content": "Search for ..."}],
    response_format={
        "type": "json_schema",
        "json_schema": {"name": "schema_name", "schema": Model.model_json_schema()},
    },
)
```

## Plain text search

```ts
// TypeScript — Vercel AI SDK
import { generateText } from "ai";

const { text } = await generateText({
  model: interfaze.chat("interfaze-beta"),
  prompt: "What is the current ...",
});
```

```python
# Python — OpenAI SDK
response = interfaze.chat.completions.create(
    model="interfaze-beta",
    messages=[{"role": "user", "content": "What is the current ..."}],
)
```

## Precontext metadata

Raw search results (`title`, `description`, `content`, `snippets`, `url` per hit) are returned on the response's `precontext` array. For structured queries, a follow-up web extract may also appear as a `precontext` entry with `name: "web_extract"`.

```ts
// TypeScript — OpenAI SDK
// @ts-expect-error precontext is not typed
const precontext = response.precontext;
console.log(precontext[0]?.result); // array of search hits
```

```ts
// TypeScript — Vercel AI SDK
const { object, response } = await generateObject({ /* ... */ });
// @ts-expect-error precontext is not typed
const precontext = response.body?.precontext;
```

```python
# Python — OpenAI SDK
precontext = getattr(response, "precontext", None)
print(precontext[0]["result"] if precontext else None)
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
- A web search is automatically triggered when the prompt requires current information.
- Combine with other skills: search the web, then use `structured-output` to normalize the results.
