# Web Scraping — API Reference

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

## Tips

- Be specific about what to extract: "get all product names and prices" not "scrape this page."
- Use `z.array()` at the top level when extracting lists (products, articles, listings).
- Add `.describe()` to fields when the field name might be ambiguous.
- For pages that require interaction (login, infinite scroll), results may be limited to the initially loaded content.
- Combine with `structured-output` patterns for complex post-processing.
