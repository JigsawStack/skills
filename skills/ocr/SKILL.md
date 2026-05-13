---
name: vocr
description: >
  Use Interfaze AI for OCR and visual text extraction from images, screenshots,
  scans, PDFs, invoices, receipts, forms, and documents with complex layouts.
  Activate when the user wants to extract text, fields, or structured data from
  an image or scanned document — even if they say "read this", "what does this say",
  or "pull the data from this photo" without explicitly mentioning OCR.
---

# VOCR — Visual OCR with Interfaze AI

Extract text and structured fields from images and scanned documents using Interfaze AI's vision OCR capability.

## When to use this skill

- The user has an image, screenshot, scan, or photo and wants text extracted from it
- The input is a receipt, invoice, ID card, form, menu, label, or any document captured as an image
- The user wants structured fields (name, address, totals, line items) pulled from a visual source
- The user says things like "read this image", "extract the text", "what does this receipt say"
- The task involves layout-aware extraction where text position matters

## When not to use this skill

- The input is a native text file, JSON, or CSV — no vision needed
- The user wants to transcribe audio — use the `speech-to-text` skill instead
- The task is purely about detecting or locating objects in an image without reading text — use the `object-detection` skill instead
- The user needs a multi-step document pipeline beyond single-image extraction — combine with `structured-output`

## Workflow

1. Confirm the input is an image URL or base64-encoded image.
2. Decide if you need plain text extraction or structured field extraction.
3. For plain text, use `generateText` with a prompt like "Extract all text from this image."
4. For structured fields, use `generateObject` with a Zod schema matching the expected output shape.
5. Pass the image in the message content array as `{ type: "image", image: "<url>" }`.
6. If the extracted data needs further structuring, combine with the `structured-output` skill.

## Setup

```ts
import { createOpenAI } from "@ai-sdk/openai";
import { generateObject } from "ai";
import z from "zod";

const interfaze = createOpenAI({
  baseURL: "https://interfaze.ai/v1",
  apiKey: process.env.INTERFAZE_API_KEY,
});
```

## Example: Extract fields from an ID card

```ts
const response = await generateObject({
  model: interfaze.chat("interfaze-beta"),
  messages: [
    {
      role: "user",
      content: [
        { type: "image", image: "https://example.com/id-card.jpg" },
        { type: "text", text: "Extract information from the image based on the schema." },
      ],
    },
  ],
  schema: z.object({
    full_first_name: z.string(),
    full_last_name: z.string(),
    full_address: z.string().nullable(),
    email: z.string().nullable(),
    id_type: z.string(),
  }),
});
```

## Example: Extract receipt line items

```ts
const response = await generateObject({
  model: interfaze.chat("interfaze-beta"),
  messages: [
    {
      role: "user",
      content: [
        { type: "image", image: "https://example.com/receipt.jpg" },
        { type: "text", text: "Extract text from the image." },
      ],
    },
  ],
  schema: z.object({
    items: z.array(z.object({ name: z.string(), price: z.string() })),
    highlighted_items: z.array(z.object({ name: z.string(), price: z.string() }))
      .describe("highlighted items in the image that is fully highlighted"),
    total_cost: z.string(),
    tax: z.string(),
  }),
});
```

## Available references

- [references/api.md](references/api.md) — API usage details for VOCR
- [references/examples.md](references/examples.md) — Additional trigger examples and patterns
