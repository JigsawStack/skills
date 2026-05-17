# Web Scraping — API Reference

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

Interfaze AI has built-in web scraping. Include a URL in your prompt and define a schema for the data you want to extract. The model fetches the page, parses the content, and returns structured data matching your schema.

No separate scraping library or browser automation is needed.

## Structured extraction

```ts
const response = await generateObject({
  model: interfaze.chat("interfaze-beta"),
  prompt: "Extract [what you need] from [URL]",
  schema: z.object({
    // Define your expected data shape
  }),
});
```

## Run task mode (raw output)

For maximum speed and lowest cost without a custom schema, set `<task>scraper</task>` in the system message. The model fetches the page and returns the scraped content in a fixed structure.

```ts
const response = await generateObject({
  model: interfaze.chat("interfaze-beta"),
  system: "<task>scraper</task>",
  schema: z.any(),
  messages: [
    {
      role: "user",
      content: "Extract post titles and points from https://news.ycombinator.com",
    },
  ],
});
```

## Tips

- Pass the URL inline in the prompt (e.g. `"Extract products from https://..."`). No separate URL parameter is needed.
- Be specific about what to extract: "get all product names and prices" not "scrape this page."
- Use `z.array()` at the top level when extracting lists (products, articles, listings).
- Add `.describe()` to fields when the field name might be ambiguous.
- Residential proxies, browser infrastructure, and bot-block handling are built in — no extra setup required.
