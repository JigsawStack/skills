---
name: ocr
description: >
  Use Interfaze AI for OCR and visual text extraction from images, screenshots,
  scans, PDFs, invoices, receipts, forms, and documents with complex layouts.
  Activate when the user wants to extract text, fields, or structured data from
  an image or scanned document — even if they say "read this", "what does this say",
  or "pull the data from this photo" without explicitly mentioning OCR.
---

# OCR — Visual OCR with Interfaze AI

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

## Workflow

1. Confirm the input is an image URL, PDF URL, or base64-encoded file.
2. Decide if you need plain text extraction or structured field extraction.
3. For structured fields, define a Zod (TypeScript) or Pydantic (Python) schema matching the expected output.
4. Pass the file in the message content array using the SDK's file/image part format.
5. Match the chosen SDK and language to the user's project.

## Setup

### TypeScript — OpenAI SDK

```ts
import OpenAI from "openai";

const interfaze = new OpenAI({
  baseURL: "https://api.interfaze.ai/v1",
  apiKey: process.env.INTERFAZE_API_KEY,
});
```

### TypeScript — Vercel AI SDK

```ts
import { createOpenAI } from "@ai-sdk/openai";

const interfaze = createOpenAI({
  baseURL: "https://api.interfaze.ai/v1",
  apiKey: process.env.INTERFAZE_API_KEY,
});
```

### TypeScript — LangChain SDK

```ts
import { ChatOpenAI } from "@langchain/openai";

const interfaze = new ChatOpenAI({
  configuration: { baseURL: "https://api.interfaze.ai/v1" },
  apiKey: process.env.INTERFAZE_API_KEY,
  model: "interfaze-beta",
});
```

### Python — OpenAI SDK

```python
from openai import OpenAI

interfaze = OpenAI(
    base_url="https://api.interfaze.ai/v1",
    api_key="<your-api-key>",
)
```

### Python — LangChain SDK

```python
from langchain_openai import ChatOpenAI

interfaze = ChatOpenAI(
    base_url="https://api.interfaze.ai/v1",
    api_key="<your-api-key>",
    model="interfaze-beta",
)
```

## Example: Extract fields from an ID card

### TypeScript — OpenAI SDK

```ts
import { z } from "zod";
import { zodResponseFormat } from "openai/helpers/zod";

const IDSchema = z.object({
  first_name: z.string().describe("First name on the ID"),
  last_name: z.string().describe("Last name on the ID"),
  dob: z.string().describe("Date of birth on the ID"),
  driver_licence_number: z.string().describe("Driver licence number on the ID"),
});

const response = await interfaze.chat.completions.create({
  model: "interfaze-beta",
  messages: [
    {
      role: "user",
      content: [
        { type: "text", text: "Extract the details from this ID" },
        { type: "image_url", image_url: { url: "https://example.com/id.jpg" } },
      ],
    },
  ],
  response_format: zodResponseFormat(IDSchema, "id_schema"),
});

console.log(response.choices[0].message.content);
```

### TypeScript — Vercel AI SDK

```ts
import { generateObject } from "ai";
import { z } from "zod";

const IDSchema = z.object({
  first_name: z.string().describe("First name on the ID"),
  last_name: z.string().describe("Last name on the ID"),
  dob: z.string().describe("Date of birth on the ID"),
  driver_licence_number: z.string().describe("Driver licence number on the ID"),
});

const { object } = await generateObject({
  model: interfaze.chat("interfaze-beta"),
  schema: IDSchema,
  messages: [
    {
      role: "user",
      content: [
        { type: "text", text: "Extract the details from this ID" },
        {
          type: "image",
          mediaType: "image/jpeg",
          image: "https://example.com/id.jpg",
        },
      ],
    },
  ],
});

console.log(object);
```

### TypeScript — LangChain SDK

```ts
import { z } from "zod";

const IDSchema = z.object({
  first_name: z.string().describe("First name on the ID"),
  last_name: z.string().describe("Last name on the ID"),
  dob: z.string().describe("Date of birth on the ID"),
  driver_licence_number: z.string().describe("Driver licence number on the ID"),
});

const structuredModel = interfaze.withStructuredOutput(IDSchema);

const response = await structuredModel.invoke([
  {
    role: "user",
    content: [
      { type: "text", text: "Extract the details from this ID" },
      { type: "image_url", image_url: { url: "https://example.com/id.jpg" } },
    ],
  },
]);

console.log(response);
```

### Python — OpenAI SDK

```python
from pydantic import BaseModel, Field

class IDSchema(BaseModel):
    first_name: str = Field(..., description="First name on the ID")
    last_name: str = Field(..., description="Last name on the ID")
    dob: str = Field(..., description="Date of birth on the ID")
    driver_licence_number: str = Field(..., description="Driver licence number on the ID")

response = interfaze.chat.completions.create(
    model="interfaze-beta",
    messages=[
        {
            "role": "user",
            "content": [
                {"type": "text", "text": "Extract the details from this ID"},
                {"type": "image_url", "image_url": {"url": "https://example.com/id.jpg"}},
            ],
        }
    ],
    response_format={
        "type": "json_schema",
        "json_schema": {"name": "id_schema", "schema": IDSchema.model_json_schema()},
    },
)

print(response.choices[0].message.content)
```

### Python — LangChain SDK

```python
from langchain_core.messages import HumanMessage
from pydantic import BaseModel, Field

class IDSchema(BaseModel):
    first_name: str = Field(..., description="First name on the ID")
    last_name: str = Field(..., description="Last name on the ID")
    dob: str = Field(..., description="Date of birth on the ID")
    driver_licence_number: str = Field(..., description="Driver licence number on the ID")

structured_llm = interfaze.with_structured_output(IDSchema)

response = structured_llm.invoke([
    HumanMessage(content=[
        {"type": "text", "text": "Extract the details from this ID"},
        {"type": "image_url", "image_url": {"url": "https://example.com/id.jpg"}},
    ])
])

print(response)
```

## Example: Extract receipt line items

### TypeScript — OpenAI SDK

```ts
import { z } from "zod";
import { zodResponseFormat } from "openai/helpers/zod";

const ReceiptSchema = z.object({
  items: z.array(z.object({ name: z.string(), price: z.string() })),
  total_cost: z.string(),
  tax: z.string(),
});

const response = await interfaze.chat.completions.create({
  model: "interfaze-beta",
  messages: [
    {
      role: "user",
      content: [
        {
          type: "text",
          text: "Extract the line items and totals from this receipt",
        },
        {
          type: "image_url",
          image_url: { url: "https://example.com/receipt.jpg" },
        },
      ],
    },
  ],
  response_format: zodResponseFormat(ReceiptSchema, "receipt_schema"),
});

console.log(response.choices[0].message.content);
```

### TypeScript — Vercel AI SDK

```ts
import { generateObject } from "ai";
import { z } from "zod";

const ReceiptSchema = z.object({
  items: z.array(z.object({ name: z.string(), price: z.string() })),
  total_cost: z.string(),
  tax: z.string(),
});

const { object } = await generateObject({
  model: interfaze.chat("interfaze-beta"),
  schema: ReceiptSchema,
  messages: [
    {
      role: "user",
      content: [
        {
          type: "text",
          text: "Extract the line items and totals from this receipt",
        },
        {
          type: "image",
          mediaType: "image/jpeg",
          image: "https://example.com/receipt.jpg",
        },
      ],
    },
  ],
});

console.log(object);
```

### TypeScript — LangChain SDK

```ts
import { z } from "zod";

const ReceiptSchema = z.object({
  items: z.array(z.object({ name: z.string(), price: z.string() })),
  total_cost: z.string(),
  tax: z.string(),
});

const structuredModel = interfaze.withStructuredOutput(ReceiptSchema);

const response = await structuredModel.invoke([
  {
    role: "user",
    content: [
      {
        type: "text",
        text: "Extract the line items and totals from this receipt",
      },
      {
        type: "image_url",
        image_url: { url: "https://example.com/receipt.jpg" },
      },
    ],
  },
]);

console.log(response);
```

### Python — OpenAI SDK

```python
from typing import List
from pydantic import BaseModel

class ReceiptItem(BaseModel):
    name: str
    price: str

class ReceiptSchema(BaseModel):
    items: List[ReceiptItem]
    total_cost: str
    tax: str

response = interfaze.chat.completions.create(
    model="interfaze-beta",
    messages=[
        {
            "role": "user",
            "content": [
                {"type": "text", "text": "Extract the line items and totals from this receipt"},
                {"type": "image_url", "image_url": {"url": "https://example.com/receipt.jpg"}},
            ],
        }
    ],
    response_format={
        "type": "json_schema",
        "json_schema": {"name": "receipt_schema", "schema": ReceiptSchema.model_json_schema()},
    },
)

print(response.choices[0].message.content)
```

### Python — LangChain SDK

```python
from typing import List
from langchain_core.messages import HumanMessage
from pydantic import BaseModel

class ReceiptItem(BaseModel):
    name: str
    price: str

class ReceiptSchema(BaseModel):
    items: List[ReceiptItem]
    total_cost: str
    tax: str

structured_llm = interfaze.with_structured_output(ReceiptSchema)

response = structured_llm.invoke([
    HumanMessage(content=[
        {"type": "text", "text": "Extract the line items and totals from this receipt"},
        {"type": "image_url", "image_url": {"url": "https://example.com/receipt.jpg"}},
    ])
])

print(response)
```

## PDF input

Pass PDFs the same way audio files are passed: as a `file` part. See [references/api.md](references/api.md) for the PDF input format across all SDKs.

## Available references

- [references/api.md](references/api.md) — API usage details, raw task mode, and PDF input
- [references/examples.md](references/examples.md) — Additional trigger examples and patterns
