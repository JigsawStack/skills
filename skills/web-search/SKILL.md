---
name: web-search
description: >
  Use Interfaze AI to search the web and retrieve current information, facts, prices,
  news, or any real-time data. Activate when the user needs up-to-date information
  from the internet, wants to look something up, or asks a question that requires
  current knowledge — even if they say "look this up", "find out about", or "what's
  the latest on" without explicitly mentioning web search.
---

# Web Search with Interfaze AI

Search the web and retrieve current information using Interfaze AI's built-in web search capability.

## When to use this skill

- The user needs current or real-time information that isn't in the model's training data
- The user asks about prices, availability, news, events, or recent developments
- The user says "search for", "look up", "find out about", "what's the latest"
- The task requires verifying facts against current web sources
- The user provides a topic and needs comprehensive, up-to-date information about it

## When not to use this skill

- The question can be answered from the model's existing knowledge
- The user wants to scrape a specific URL for structured data — use `web-scraping`
- The user has an image or audio to process — use `ocr`, `object-detection`, or `speech-to-text`
- The user wants to extract data from a specific known page — use `web-scraping`

## Workflow

1. Identify what information the user needs from the web.
2. Formulate a clear search prompt that describes what to find.
3. Use `generateObject` with a schema matching the expected result shape, or `generateText` for a plain summary.
4. Interfaze handles the web search and returns results directly.
5. Return the findings in the format the user requested.

## Setup

```ts
import { createOpenAI } from "@ai-sdk/openai";
import { generateObject, generateText } from "ai";
import z from "zod";

const interfaze = createOpenAI({
  baseURL: "https://api.interfaze.ai/v1",
  apiKey: process.env.INTERFAZE_API_KEY,
});
```

## Example: Search and extract structured results

```ts
const response = await generateObject({
  model: interfaze.chat("interfaze-beta"),
  prompt: "Search for the current price of Bitcoin and Ethereum.",
  schema: z.object({
    results: z.array(
      z.object({
        name: z.string(),
        price_usd: z.string(),
        change_24h: z.string().nullable(),
      })
    ),
  }),
});
```

## Example: Research a topic

```ts
const response = await generateText({
  model: interfaze.chat("interfaze-beta"),
  prompt: "What are the latest developments in quantum computing as of 2026?",
});

console.log(response.text);
```

## Example: Look up a person or company

```ts
const response = await generateObject({
  model: interfaze.chat("interfaze-beta"),
  prompt: "Find information about Acme Corp from their website and LinkedIn.",
  schema: z.object({
    company_name: z.string(),
    description: z.string(),
    headquarters: z.string().nullable(),
    industry: z.string(),
    key_people: z.array(z.string()),
  }),
});
```

## Available references

- [references/api.md](references/api.md) — API usage details for web search
- [references/examples.md](references/examples.md) — Additional trigger examples and patterns
