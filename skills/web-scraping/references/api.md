# Web Scraping — API Reference

## Endpoint

```
POST https://api.interfaze.ai/v1/chat/completions
```

Model name: `interfaze-beta`.

## How it works

Interfaze AI has built-in web scraping with:

- Residential proxies that automatically rotate when needed and handle bot blocks
- AI human-behavior simulation and interactions
- Auto-scaling browser infrastructure with no concurrency limits

Include a URL in the prompt and define a schema for the data you want. No separate scraping library is required.

## Structured extraction

The URL goes inline in the prompt (no separate parameter). Provide a Zod (TypeScript) or Pydantic (Python) schema describing the desired output.

```ts
// TypeScript — Vercel AI SDK
const { object } = await generateObject({
  model: interfaze.chat("interfaze-beta"),
  schema: z.object({ ... }),
  prompt: "Extract [what you need] from [URL]",
});
```

```python
# Python — OpenAI SDK
response = interfaze.chat.completions.create(
    model="interfaze-beta",
    messages=[{"role": "user", "content": "Extract [what you need] from [URL]"}],
    response_format={
        "type": "json_schema",
        "json_schema": {"name": "schema_name", "schema": MyModel.model_json_schema()},
    },
)
```

## Run task mode (raw output)

For maximum speed and lowest cost without a custom schema, set `<task>scraper</task>` in the system message. The model fetches the page and returns scraped content in a fixed structure on `precontext`.

### TypeScript — OpenAI SDK

```ts
import { z } from "zod";
import { zodResponseFormat } from "openai/helpers/zod";

const response = await interfaze.chat.completions.create({
  model: "interfaze-beta",
  messages: [
    { role: "system", content: "<task>scraper</task>" },
    { role: "user", content: "Extract post titles and points from https://news.ycombinator.com" },
  ],
  response_format: zodResponseFormat(z.any(), "scraper_schema"),
});
```

### TypeScript — Vercel AI SDK

```ts
import { generateObject } from "ai";
import { z } from "zod";

const { object } = await generateObject({
  model: interfaze.chat("interfaze-beta"),
  system: "<task>scraper</task>",
  schema: z.any(),
  messages: [
    { role: "user", content: "Extract post titles and points from https://news.ycombinator.com" },
  ],
});
```

### TypeScript — LangChain SDK

```ts
const structuredModel = interfaze.withStructuredOutput({});

const response = await structuredModel.invoke([
  { role: "system", content: "<task>scraper</task>" },
  { role: "user", content: "Extract post titles and points from https://news.ycombinator.com" },
]);
```

### Python — OpenAI SDK

```python
response = interfaze.chat.completions.create(
    model="interfaze-beta",
    messages=[
        {"role": "system", "content": "<task>scraper</task>"},
        {
            "role": "user",
            "content": "Extract post titles and points from https://news.ycombinator.com",
        },
    ],
    response_format={
        "type": "json_schema",
        "json_schema": {
            "name": "scraper_schema",
            "schema": {"type": "object", "properties": {}, "additionalProperties": True},
        },
    },
)
```

### Python — LangChain SDK

```python
from langchain_core.messages import SystemMessage, HumanMessage

structured = interfaze.with_structured_output({
    "name": "scraper_schema",
    "schema": {"type": "object", "properties": {}, "additionalProperties": True},
})

response = structured.invoke([
    SystemMessage(content="<task>scraper</task>"),
    HumanMessage(content="Extract post titles and points from https://news.ycombinator.com"),
])
```

## Tips

- Pass the URL inline in the prompt. No separate URL parameter is needed.
- Be specific about what to extract: "get all product names and prices" works better than "scrape this page".
- Use `z.array(...)` / `List[...]` at the top level when extracting lists (products, articles, listings).
- Use `.describe()` / `Field(description=...)` for ambiguous field names.
- Residential proxies, browser infrastructure, and bot-block handling are built in — no extra setup required.
