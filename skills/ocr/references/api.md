# VOCR — API Reference

## Endpoint

Interfaze AI uses an OpenAI-compatible chat completions endpoint:

```
POST https://interfaze.ai/v1/chat/completions
```

## Authentication

Pass your API key via the `Authorization` header or configure it through the SDK:

```ts
const interfaze = createOpenAI({
  baseURL: "https://interfaze.ai/v1",
  apiKey: process.env.INTERFAZE_API_KEY,
});
```

## Image input format

Images are passed in the message content array:

```ts
{
  role: "user",
  content: [
    { type: "image", image: "<url-or-base64>" },
    { type: "text", text: "Your extraction prompt here." },
  ],
}
```

Supported image sources:
- Public URLs (HTTPS)
- Base64-encoded image data

## Structured extraction

Use `generateObject` with a Zod schema to get typed, structured output:

```ts
const response = await generateObject({
  model: interfaze.chat("interfaze-beta"),
  messages: [...],
  schema: z.object({
    // Define your expected output shape
  }),
});

// response.object contains the typed result
```

## Plain text extraction

Use `generateText` when you only need raw text:

```ts
const response = await generateText({
  model: interfaze.chat("interfaze-beta"),
  messages: [
    {
      role: "user",
      content: [
        { type: "image", image: "<url>" },
        { type: "text", text: "Extract all text from this image." },
      ],
    },
  ],
});

// response.text contains the extracted text
```

## Tips

- Use `.describe()` on Zod fields when the field name alone is ambiguous.
- For multilingual text, specify the target language in the prompt or schema field descriptions.
- For layout-aware extraction with bounding boxes, include coordinate fields in your schema.
