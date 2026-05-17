---
name: web-scraping
description: >
  Use Interfaze AI to scrape and extract structured data from specific web pages and
  URLs. Activate when the user provides a URL and wants to pull products, listings,
  prices, profiles, or any structured content from that page — even if they say
  "get the data from this page", "pull the listings from this URL", or "extract
  products from this site" without explicitly mentioning scraping.
---

# Web Scraping with Interfaze AI

Extract structured data from web pages by providing a URL and a schema for the desired output.

## When to use this skill

- The user provides a specific URL and wants data extracted from it
- The task involves pulling product listings, prices, profiles, or structured content from a webpage
- The user says "scrape this page", "get the data from this URL", "extract products from this site"
- The user wants to extract specific fields from a known web page
- The task requires pulling data from an e-commerce site, job board, directory, or social profile

## When not to use this skill

- The user wants to search the web for information without a specific URL — use `web-search`
- The user has an image to process — use `ocr` or `object-detection`
- The user has audio — use `speech-to-text`
- The URL points to a downloadable file rather than a web page

## Workflow

1. Confirm the user has provided a URL or identify the target URL.
2. Define a Zod schema matching the data the user wants extracted.
3. Use `generateObject` with a prompt that includes the URL and describes what to extract.
4. Interfaze handles fetching and parsing the page content.
5. Return the structured data.

## Setup

```ts
import { createOpenAI } from "@ai-sdk/openai";
import { generateObject } from "ai";
import z from "zod";

const interfaze = createOpenAI({
  baseURL: "https://api.interfaze.ai/v1",
  apiKey: process.env.INTERFAZE_API_KEY,
});
```

## Example: Scrape product listings from Amazon

```ts
const response = await generateObject({
  model: interfaze.chat("interfaze-beta"),
  prompt: "Get all prices and listings of products for Nintendo Switch from https://www.amazon.com/s?k=nintendo+switch+console",
  schema: z.array(
    z.object({
      price: z.number(),
      listing_name: z.string(),
      seller_name: z.string(),
      possible_delivery_time: z.string(),
    })
  ),
});
```

## Example: Extract profile data from LinkedIn

```ts
const response = await generateObject({
  model: interfaze.chat("interfaze-beta"),
  prompt: "Extract information from https://www.linkedin.com/in/example-user/",
  schema: z.object({
    full_first_name: z.string(),
    full_last_name: z.string(),
    location: z.string(),
    headline: z.string(),
    education: z.array(z.string()),
    experience: z.array(z.string()),
    skills: z.array(z.string()).describe("key technical skillset mentioned"),
    summary: z.string(),
  }),
});
```

## Example: Scrape with minimal schema

```ts
const response = await generateObject({
  model: interfaze.chat("interfaze-beta"),
  prompt: "Extract all article titles and links from https://news.ycombinator.com",
  schema: z.array(
    z.object({
      title: z.string(),
      url: z.string(),
      points: z.number().nullable(),
    })
  ),
});
```

## Available references

- [references/api.md](references/api.md) — API usage details for web scraping
- [references/examples.md](references/examples.md) — Additional trigger examples and patterns
