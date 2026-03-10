# Structured Output — Schema Patterns

## Basic object

```ts
z.object({
  title: z.string(),
  summary: z.string(),
  confidence: z.number(),
})
```

## Array of objects

```ts
z.array(
  z.object({
    name: z.string(),
    value: z.string(),
  })
)
```

## Nullable fields

Use when a field may not exist in the source data:

```ts
z.object({
  name: z.string(),
  email: z.string().nullable(),
  phone: z.string().nullable(),
})
```

## Described fields

Use `.describe()` when the field name alone is ambiguous:

```ts
z.object({
  items: z.array(
    z.object({
      name: z.string(),
      price: z.string(),
    })
  ),
  highlighted_items: z.array(
    z.object({
      name: z.string(),
      price: z.string(),
    })
  ).describe("items that are visually highlighted or marked in the source"),
})
```

## Nested coordinates (for spatial extraction)

```ts
z.object({
  text_in_original_language: z.string(),
  bounds: z.array(
    z.object({
      top_left_x: z.number(),
      top_left_y: z.number(),
      bottom_right_x: z.number(),
      bottom_right_y: z.number(),
    })
  ).describe("bounding box for each line"),
})
```

## Response expectations

- `generateObject` returns `{ object: <typed result> }` that matches your schema exactly.
- Invalid responses are automatically retried by the Vercel AI SDK.
- Keep schemas reasonably sized. Split large extractions into multiple calls if needed.
