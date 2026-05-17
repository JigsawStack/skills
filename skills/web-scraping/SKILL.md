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

Extract structured data from web pages by including a URL in the prompt and defining a schema for the desired output. Interfaze has built-in residential proxies, browser infrastructure, and bot-block handling.

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
2. Define a schema (Zod for TypeScript, Pydantic for Python) matching the data the user wants extracted.
3. Include the URL inline in the prompt; the model fetches and parses the page.
4. Return the structured data.

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

## Example: Scrape product listings

### TypeScript — OpenAI SDK

```ts
import { z } from "zod";
import { zodResponseFormat } from "openai/helpers/zod";

const ProductSchema = z.object({
  price: z.number(),
  listing_name: z.string(),
});

const response = await interfaze.chat.completions.create({
  model: "interfaze-beta",
  messages: [
    {
      role: "user",
      content:
        "Extract the information from https://www.amazon.com/Nintendo-Switch-Neon-Blue-Joy-Con/dp/B0BFJWCYTL",
    },
  ],
  response_format: zodResponseFormat(ProductSchema, "product_schema"),
});

console.log(response.choices[0].message.content);
```

### TypeScript — Vercel AI SDK

```ts
import { generateObject } from "ai";
import { z } from "zod";

const ProductSchema = z.object({
  price: z.number(),
  listing_name: z.string(),
});

const { object } = await generateObject({
  model: interfaze.chat("interfaze-beta"),
  schema: ProductSchema,
  prompt:
    "Extract the information from https://www.amazon.com/Nintendo-Switch-Neon-Blue-Joy-Con/dp/B0BFJWCYTL",
});
```

### TypeScript — LangChain SDK

```ts
import { z } from "zod";

const ProductSchema = z.object({
  price: z.number(),
  listing_name: z.string(),
});

const structuredModel = interfaze.withStructuredOutput(ProductSchema);

const response = await structuredModel.invoke(
  "Extract the information from https://www.amazon.com/Nintendo-Switch-Neon-Blue-Joy-Con/dp/B0BFJWCYTL"
);
```

### Python — OpenAI SDK

```python
from pydantic import BaseModel

class ProductSchema(BaseModel):
    price: float
    listing_name: str

response = interfaze.chat.completions.create(
    model="interfaze-beta",
    messages=[
        {
            "role": "user",
            "content": "Extract the information from https://www.amazon.com/Nintendo-Switch-Neon-Blue-Joy-Con/dp/B0BFJWCYTL",
        }
    ],
    response_format={
        "type": "json_schema",
        "json_schema": {"name": "product_schema", "schema": ProductSchema.model_json_schema()},
    },
)

print(response.choices[0].message.content)
```

### Python — LangChain SDK

```python
from pydantic import BaseModel

class ProductSchema(BaseModel):
    price: float
    listing_name: str

structured_llm = interfaze.with_structured_output(ProductSchema)

response = structured_llm.invoke(
    "Extract the information from https://www.amazon.com/Nintendo-Switch-Neon-Blue-Joy-Con/dp/B0BFJWCYTL"
)
```

## Example: Scrape a social profile

### TypeScript — Vercel AI SDK

```ts
import { generateObject } from "ai";
import { z } from "zod";

const LinkedInProfileSchema = z.object({
  first_name: z.string(),
  last_name: z.string(),
  location: z.string(),
  latest_education: z.string(),
  current_job: z.string(),
  followers: z.number(),
});

const { object } = await generateObject({
  model: interfaze.chat("interfaze-beta"),
  schema: LinkedInProfileSchema,
  prompt: "Extract information from https://www.linkedin.com/in/example-user/",
});
```

### Python — OpenAI SDK

```python
from pydantic import BaseModel

class LinkedInProfileSchema(BaseModel):
    first_name: str
    last_name: str
    location: str
    latest_education: str
    current_job: str
    followers: int

response = interfaze.chat.completions.create(
    model="interfaze-beta",
    messages=[
        {"role": "user", "content": "Extract information from https://www.linkedin.com/in/example-user/"},
    ],
    response_format={
        "type": "json_schema",
        "json_schema": {"name": "linkedin_schema", "schema": LinkedInProfileSchema.model_json_schema()},
    },
)
```

Other SDK variants follow the product example pattern with the schema swapped in.

## Example: Scrape an array of listings

### TypeScript — Vercel AI SDK

```ts
import { generateObject } from "ai";
import { z } from "zod";

const { object } = await generateObject({
  model: interfaze.chat("interfaze-beta"),
  schema: z.array(
    z.object({
      title: z.string(),
      url: z.string(),
      points: z.number().nullable(),
    })
  ),
  prompt: "Extract all article titles, URLs, and points from https://news.ycombinator.com",
});
```

## Raw scraping output (faster)

For maximum speed and lowest cost without a custom schema, use `<task>scraper</task>` in the system message. See [references/api.md](references/api.md) for the full multi-SDK breakdown.

## Available references

- [references/api.md](references/api.md) — API usage details and raw task mode
- [references/examples.md](references/examples.md) — Additional trigger examples and patterns
