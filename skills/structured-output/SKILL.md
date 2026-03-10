---
name: structured-output
description: >
  Use Interfaze AI to produce strict JSON or schema-constrained structured objects
  from any prompt. Activate when the user needs typed, machine-readable output —
  JSON objects, arrays, validated fields, or Zod/Pydantic-style extraction. Use this
  when the task requires a specific output shape, data normalization, or converting
  unstructured text into structured records, even if the user just says "give me this
  as JSON" or "parse this into fields."
---

# Structured Output with Interfaze AI

Generate outputs into strict, schema-validated JSON using Zod schemas and the Vercel AI SDK for any task.

## When to use this skill

- The user needs output in a specific JSON shape or typed format
- The task requires extracting structured fields from unstructured text
- The user says "give me JSON", "parse this into an object", "extract these fields"
- You need to validate AI output against a schema before using it downstream
- The output will be consumed by code, not displayed to a human

## When not to use this skill

- The user wants free-form text, summaries, or conversational responses
- The input is an image that needs OCR first — use `vocr`, then apply structured output
- The input is audio — use `speech-to-text` first, then apply structured output if needed

## Workflow

1. Identify the target output shape from the user's request.
2. Define a Zod schema that matches the expected structure.
3. Use `generateObject` with the schema to get validated output.
4. The response is typed and validated — use `response.object` directly.

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

## Example: Extract structured data from a prompt

```ts
const response = await generateObject({
  model: interfaze.chat("interfaze-beta"),
  prompt: "Write a Python script to check CPU type using the subprocess module.",
  schema: z.object({
    code: z.string(),
    sample_input: z.string(),
    sample_output: z.string(),
  }),
});

console.log(response.object);
// { code: "...", sample_input: "...", sample_output: "..." }
```

## Example: Array of structured objects

```ts
const response = await generateObject({
  model: interfaze.chat("interfaze-beta"),
  prompt: "List the top 5 programming languages by popularity.",
  schema: z.array(
    z.object({
      name: z.string(),
      rank: z.number(),
      primary_use_case: z.string(),
    })
  ),
});
```

## Example: Combined with image input (VOCR + structured output)

```ts
const response = await generateObject({
  model: interfaze.chat("interfaze-beta"),
  messages: [
    {
      role: "user",
      content: [
        { type: "image", image: "https://example.com/document.jpg" },
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

## Schema design tips

- Use `.nullable()` for fields that may not be present in the source.
- Use `.describe()` to clarify ambiguous fields — the model reads these descriptions.
- Use `z.array()` at the top level when extracting lists of items.
- Keep schemas focused. Prefer multiple targeted calls over one massive schema.

## Available references

- [references/schemas.md](references/schemas.md) — Common schema patterns
- [references/examples.md](references/examples.md) — Additional trigger examples
