# Object Detection — API Reference

## Endpoint

```
POST https://api.interfaze.ai/v1/chat/completions
```

## Authentication

```ts
const interfaze = createOpenAI({
  baseURL: "https://api.interfaze.ai/v1",
  apiKey: process.env.INTERFAZE_API_KEY,
});
```

## Image input format

Same as OCR — images are passed in the message content array:

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

## Run task mode (raw output)

For maximum speed and lowest cost when you don't need a custom schema, set `<task>object_detection</task>` in the system message. The model returns a fixed structure with `detected_objects` (each with `bounds` and `label`) and `gui_elements`.

```ts
const response = await generateObject({
  model: interfaze.chat("interfaze-beta"),
  system: "<task>object_detection</task>",
  schema: z.any(),
  messages: [
    {
      role: "user",
      content: [
        { type: "text", text: "Detect objects in this image" },
        { type: "image", image: "<url>" },
      ],
    },
  ],
});
```

## Tips

- Be specific in the prompt about what to detect. "Find all vehicles" works better than "detect objects."
- Use `.describe()` on the name field to guide the model on how to label detected objects.
- Add a `category` field to the schema if you need objects grouped by type.
- Bounding box coordinates are returned in pixels relative to the original image dimensions.
