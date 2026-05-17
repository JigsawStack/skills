# Structured Output — Schema Patterns

## Basic object

### TypeScript (Zod)

```ts
z.object({
  title: z.string(),
  summary: z.string(),
  confidence: z.number(),
})
```

### Python (Pydantic)

```python
from pydantic import BaseModel

class Item(BaseModel):
    title: str
    summary: str
    confidence: float
```

## Array of objects

### TypeScript (Zod)

```ts
z.array(
  z.object({
    name: z.string(),
    value: z.string(),
  })
)
```

### Python (Pydantic)

```python
from typing import List
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    value: str

class ItemsSchema(BaseModel):
    items: List[Item]
```

## Nullable fields

Use when a field may not exist in the source data.

### TypeScript (Zod)

```ts
z.object({
  name: z.string(),
  email: z.string().nullable(),
  phone: z.string().nullable(),
})
```

### Python (Pydantic)

```python
from typing import Optional
from pydantic import BaseModel

class Contact(BaseModel):
    name: str
    email: Optional[str] = None
    phone: Optional[str] = None
```

## Described fields

Use `.describe()` (Zod) or `Field(description=...)` (Pydantic) when the field name alone is ambiguous.

### TypeScript (Zod)

```ts
z.object({
  items: z.array(z.object({ name: z.string(), price: z.string() })),
  highlighted_items: z.array(
    z.object({ name: z.string(), price: z.string() })
  ).describe("items that are visually highlighted or marked in the source"),
})
```

### Python (Pydantic)

```python
from typing import List
from pydantic import BaseModel, Field

class Item(BaseModel):
    name: str
    price: str

class ReceiptSchema(BaseModel):
    items: List[Item]
    highlighted_items: List[Item] = Field(
        ..., description="items that are visually highlighted or marked in the source"
    )
```

## Nested coordinates (for spatial extraction)

### TypeScript (Zod)

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

### Python (Pydantic)

```python
from typing import List
from pydantic import BaseModel, Field

class Bound(BaseModel):
    top_left_x: float
    top_left_y: float
    bottom_right_x: float
    bottom_right_y: float

class SpatialSchema(BaseModel):
    text_in_original_language: str
    bounds: List[Bound] = Field(..., description="bounding box for each line")
```

## Response expectations

- All SDKs return an object that matches your schema exactly.
- Vercel AI SDK: `const { object } = await generateObject({ schema, ... })`.
- OpenAI SDK: parse `response.choices[0].message.content` as JSON, or use the SDK's helper to deserialize via the schema.
- LangChain SDK: the call returns the typed object directly.
- Invalid responses are automatically retried by the Vercel AI SDK.
- Keep schemas reasonably sized. Split large extractions into multiple calls if needed.
