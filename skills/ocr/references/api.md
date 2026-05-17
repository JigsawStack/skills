# OCR — API Reference

## Endpoint

Interfaze AI uses an OpenAI-compatible chat completions endpoint:

```
POST https://api.interfaze.ai/v1/chat/completions
```

## Authentication

Pass your API key via the `Authorization` header or configure it through the SDK:

```ts
const interfaze = createOpenAI({
  baseURL: "https://api.interfaze.ai/v1",
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

## PDF and document input

PDFs are passed the same way images are, via the `file` part:

```ts
{
  role: "user",
  content: [
    { type: "file", data: "<pdf-url>", mediaType: "application/pdf" },
    { type: "text", text: "Extract the text and tables from this document." },
  ],
}
```

## Run task mode (raw output)

For raw OCR output (no custom schema), set `<task>ocr</task>` in the system message. The response contains `extracted_text` and `sections` with line-level bounding boxes and per-word confidence scores.

```ts
const response = await generateObject({
  model: interfaze.chat("interfaze-beta"),
  system: "<task>ocr</task>",
  schema: z.any(),
  messages: [
    {
      role: "user",
      content: [
        { type: "text", text: "Extract all text from this image" },
        { type: "image", image: "<url>" },
      ],
    },
  ],
});
```

## Precontext metadata

When using the OCR capability through `generateObject`, the response also includes a `precontext` array with raw OCR metadata (bounding boxes, confidence scores). Access it via `response.body?.precontext`.

## Tips

- Use `.describe()` on Zod fields when the field name alone is ambiguous.
- For multilingual text, specify the target language in the prompt or schema field descriptions.
- For layout-aware extraction with bounding boxes, include coordinate fields in your schema.
- 100+ languages including mixed-language documents are supported.
