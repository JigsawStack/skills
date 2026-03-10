# Object Detection — API Reference

## Endpoint

```
POST https://interfaze.ai/v1/chat/completions
```

## Authentication

```ts
const interfaze = createOpenAI({
  baseURL: "https://interfaze.ai/v1",
  apiKey: process.env.INTERFAZE_API_KEY,
});
```

## Image input format

Same as VOCR — images are passed in the message content array:

```ts
{
  role: "user",
  content: [
    { type: "image", image: "<url-or-base64>" },
    { type: "text", text: "Detect objects in this image." },
  ],
}
```

## Bounding box schema

Object detection uses bounding box coordinates. The standard pattern:

```ts
z.object({
  objects: z.array(
    z.object({
      name: z.string().describe("description of the detected object"),
      top_left_x: z.number(),
      top_left_y: z.number(),
      bottom_right_x: z.number(),
      bottom_right_y: z.number(),
    })
  ),
})
```

Coordinates are returned in pixel values relative to the image dimensions.

## Combining objects and text

To detect both objects and text with positions:

```ts
z.object({
  objects: z.array(z.object({
    name: z.string(),
    top_left_x: z.number(),
    top_left_y: z.number(),
    bottom_right_x: z.number(),
    bottom_right_y: z.number(),
  })),
  texts: z.array(z.object({
    text: z.string(),
    top_left_x: z.number(),
    top_left_y: z.number(),
    bottom_right_x: z.number(),
    bottom_right_y: z.number(),
  })).describe("any text visible in the image"),
})
```

## Tips

- Be specific in the prompt about what to detect. "Find all vehicles" works better than "detect objects."
- Use `.describe()` on the name field to guide the model on how to label detected objects.
- Add a `category` field to the schema if you need objects grouped by type.
