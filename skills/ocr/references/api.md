# OCR — API Reference

## Endpoint

Interfaze AI uses an OpenAI-compatible chat completions endpoint:

```
POST https://api.interfaze.ai/v1/chat/completions
```

Model name: `interfaze-beta`.

## Image input format

### TypeScript — Vercel AI SDK

```ts
{
  role: "user",
  content: [
    { type: "text", text: "..." },
    { type: "image", mediaType: "image/jpeg", image: "<url-or-base64>" },
  ],
}
```

### TypeScript — OpenAI SDK / LangChain SDK

```ts
{
  role: "user",
  content: [
    { type: "text", text: "..." },
    { type: "image_url", image_url: { url: "<url>" } },
  ],
}
```

### Python — OpenAI SDK / LangChain SDK

```python
{
    "role": "user",
    "content": [
        {"type": "text", "text": "..."},
        {"type": "image_url", "image_url": {"url": "<url>"}},
    ],
}
```

Supported image sources: public HTTPS URLs and base64-encoded data.

## PDF and document input

### TypeScript — Vercel AI SDK

```ts
{
  role: "user",
  content: [
    { type: "text", text: "Extract the text and tables from this document" },
    { type: "file", data: "<pdf-url>", mediaType: "application/pdf" },
  ],
}
```

### TypeScript — OpenAI SDK / LangChain SDK

```ts
{
  role: "user",
  content: [
    { type: "text", text: "Extract the text and tables from this document" },
    {
      type: "file",
      file: {
        filename: "document.pdf",
        file_data: "<url-or-base64>",
      },
    },
  ],
}
```

### Python — OpenAI SDK / LangChain SDK

```python
{
    "role": "user",
    "content": [
        {"type": "text", "text": "Extract the text and tables from this document"},
        {
            "type": "file",
            "file": {
                "filename": "document.pdf",
                "file_data": "<url-or-base64>",
            },
        },
    ],
}
```

## Run task mode (raw OCR output)

For raw OCR output (no custom schema), set `<task>ocr</task>` in the system message. The response's `precontext` contains `extracted_text` and `sections` with line-level bounding boxes and per-word confidence scores.

### TypeScript — OpenAI SDK

```ts
import { z } from "zod";
import { zodResponseFormat } from "openai/helpers/zod";

const response = await interfaze.chat.completions.create({
  model: "interfaze-beta",
  messages: [
    { role: "system", content: "<task>ocr</task>" },
    {
      role: "user",
      content: [
        { type: "text", text: "Extract all text from this image" },
        { type: "image_url", image_url: { url: "<url>" } },
      ],
    },
  ],
  response_format: zodResponseFormat(z.any(), "empty_schema"),
});
```

### TypeScript — Vercel AI SDK

```ts
import { generateObject } from "ai";
import { z } from "zod";

const { object } = await generateObject({
  model: interfaze.chat("interfaze-beta"),
  system: "<task>ocr</task>",
  schema: z.any(),
  messages: [
    {
      role: "user",
      content: [
        { type: "text", text: "Extract all text from this image" },
        { type: "image", mediaType: "image/jpeg", image: "<url>" },
      ],
    },
  ],
});
```

### Python — OpenAI SDK

```python
response = interfaze.chat.completions.create(
    model="interfaze-beta",
    messages=[
        {"role": "system", "content": "<task>ocr</task>"},
        {
            "role": "user",
            "content": [
                {"type": "text", "text": "Extract all text from this image"},
                {"type": "image_url", "image_url": {"url": "<url>"}},
            ],
        },
    ],
    response_format={
        "type": "json_schema",
        "json_schema": {
            "name": "empty_schema",
            "schema": {"type": "object", "properties": {}, "additionalProperties": True},
        },
    },
)
```

## Precontext metadata

For non-task calls, raw OCR results (bounding boxes, per-word confidence scores) are returned on `response.precontext` (OpenAI SDK / Python) or `response.body?.precontext` (Vercel AI SDK).

```ts
// @ts-expect-error precontext is not typed
const precontext = response.precontext ?? response.body?.precontext;
console.log("OCR Results:", precontext[0]?.result);
```

## Plain text extraction

Use `generateText` (Vercel AI SDK) or omit `response_format` (OpenAI / LangChain SDK) when you only need raw text. Example for the Vercel AI SDK:

```ts
import { generateText } from "ai";

const { text } = await generateText({
  model: interfaze.chat("interfaze-beta"),
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

## Tips

- Use `.describe()` / `Field(description=...)` when the field name alone is ambiguous.
- For multilingual text, specify the target language in the prompt or schema field descriptions.
- For layout-aware extraction with bounding boxes, include coordinate fields in your schema.
- 100+ languages including mixed-language documents are supported.
