---
name: object-detection
description: >
  Use Interfaze AI to detect, locate, and identify objects in images and PDFs with
  bounding box coordinates. Activate when the user wants to find objects, get their
  positions, count items, or locate specific things in an image — even if they say
  "find the car in this photo", "where is the logo", or "what objects are in this
  image" without explicitly mentioning object detection.
---

# Object Detection with Interfaze AI

Detect and locate objects in images with bounding box coordinates using Interfaze AI.

## When to use this skill

- The user wants to find or locate objects in an image
- The user needs bounding box coordinates for detected objects
- The user wants to count specific items in a photo
- The user says "find the X in this image", "where is the Y", "what objects are here"
- The task involves spatial understanding — positions, regions, or layout of objects
- The user wants both object positions and any visible text with coordinates

## When not to use this skill

- The user only wants to read text from an image — use `vocr`
- The user wants to transcribe audio — use `speech-to-text`
- The input is not an image
- The user wants image classification without location data

## Workflow

1. Confirm the input is an image URL or base64-encoded image.
2. Determine what objects the user wants to detect, or if they want all objects.
3. Build a Zod schema with object names and bounding box coordinate fields.
4. Use `generateObject` with the image and a prompt describing what to detect.
5. Return the detected objects with their positions.

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

## Example: Detect objects and text in an image

```ts
const response = await generateObject({
  model: interfaze.chat("interfaze-beta"),
  messages: [
    {
      role: "user",
      content: [
        { type: "image", image: "https://example.com/photo.jpg" },
        { type: "text", text: "Get the position of the crane in the image and any text" },
      ],
    },
  ],
  schema: z.object({
    objects: z.array(
      z.object({
        name: z.string().describe("describe the object in the image"),
        top_left_x: z.number(),
        top_left_y: z.number(),
        bottom_right_x: z.number(),
        bottom_right_y: z.number(),
      })
    ),
    texts: z.array(
      z.object({
        text: z.string(),
        top_left_x: z.number(),
        top_left_y: z.number(),
        bottom_right_x: z.number(),
        bottom_right_y: z.number(),
      })
    ).describe("any alphabetic characters text in the image"),
  }),
});
```

## Example: Detect specific objects

```ts
const response = await generateObject({
  model: interfaze.chat("interfaze-beta"),
  messages: [
    {
      role: "user",
      content: [
        { type: "image", image: "https://example.com/street.jpg" },
        { type: "text", text: "Detect all vehicles and pedestrians in this image." },
      ],
    },
  ],
  schema: z.object({
    objects: z.array(
      z.object({
        name: z.string(),
        category: z.string().describe("vehicle or pedestrian"),
        top_left_x: z.number(),
        top_left_y: z.number(),
        bottom_right_x: z.number(),
        bottom_right_y: z.number(),
      })
    ),
  }),
});
```

## Available references

- [references/api.md](references/api.md) — API usage details for object detection
- [references/examples.md](references/examples.md) — Additional trigger examples and patterns
