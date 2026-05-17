# Speech to Text — API Reference

## Endpoint

```
POST https://api.interfaze.ai/v1/chat/completions
```

Model name: `interfaze-beta`.

## Audio input format

### TypeScript — Vercel AI SDK

```ts
{
  role: "user",
  content: [
    { type: "text", text: "Transcribe this audio" },
    { type: "file", data: "<audio-url>", mediaType: "audio/mpeg" },
  ],
}
```

### TypeScript — OpenAI SDK / LangChain SDK

```ts
{
  role: "user",
  content: [
    { type: "text", text: "Transcribe this audio" },
    {
      type: "file",
      file: {
        filename: "audio.mp3",
        file_data: "<url-or-base64>",
      },
    },
  ],
}
```

### Python — OpenAI SDK / LangChain SDK

```python
{
    "role": "user",
    "content": [
        {"type": "text", "text": "Transcribe this audio"},
        {
            "type": "file",
            "file": {
                "filename": "audio.mp3",
                "file_data": "<url-or-base64>",
            },
        },
    ],
}
```

### Inline URL in prompt

Across all SDKs you can also include the URL directly in the prompt text. This is slightly faster than the file part:

```ts
{ role: "user", content: "Transcribe https://example.com/audio.mp3" }
```

## MIME types and supported formats

- MP3 → `audio/mpeg`
- WAV → `audio/wav`
- M4A → `audio/mp4`
- Other common formats are also supported via their canonical IANA MIME types.

## Schemas

### Basic transcription

```ts
// TypeScript (Zod)
z.object({ text: z.string() })
```

```python
# Python (Pydantic)
class STTSchema(BaseModel):
    text: str
```

### Speaker diarization

`speaker_id` is a string. Up to 50 speakers are supported.

```ts
// TypeScript (Zod)
z.object({
  full_text: z.string(),
  chunks: z.array(
    z.object({
      speaker_id: z.string(),
      text: z.string(),
      start_time: z.number(),
      end_time: z.number(),
    })
  ),
  number_of_speakers: z.number(),
})
```

```python
# Python (Pydantic)
from typing import List

class Chunk(BaseModel):
    speaker_id: str
    text: str
    start_time: float
    end_time: float

class DiarizationSchema(BaseModel):
    full_text: str
    chunks: List[Chunk]
    number_of_speakers: int
```

Extend each chunk with analysis fields like `sentiment: z.enum(["positive", "negative", "neutral"])` for sentiment per speaker.

## Run task mode (raw output)

Set `<task>speech_to_text</task>` in the system message. Response contains `text` and `chunks: [{ timestamp: [start, end], text }]`.

### TypeScript — OpenAI SDK

```ts
import { z } from "zod";
import { zodResponseFormat } from "openai/helpers/zod";

const response = await interfaze.chat.completions.create({
  model: "interfaze-beta",
  messages: [
    { role: "system", content: "<task>speech_to_text</task>" },
    {
      role: "user",
      content: [
        { type: "text", text: "Transcribe https://example.com/audio.mp3" },
      ],
    },
  ],
  response_format: zodResponseFormat(z.any(), "empty_schema"),
});
```

### TypeScript — Vercel AI SDK

```ts
import { generateObject } from "ai";
import { z } from "zod";

const { object } = await generateObject({
  model: interfaze.chat("interfaze-beta"),
  system: "<task>speech_to_text</task>",
  schema: z.any(),
  messages: [
    {
      role: "user",
      content: [{ type: "text", text: "Transcribe https://example.com/audio.mp3" }],
    },
  ],
});
```

### Python — OpenAI SDK

```python
response = interfaze.chat.completions.create(
    model="interfaze-beta",
    messages=[
        {"role": "system", "content": "<task>speech_to_text</task>"},
        {
            "role": "user",
            "content": [
                {"type": "text", "text": "Transcribe https://example.com/audio.mp3"},
            ],
        },
    ],
    response_format={
        "type": "json_schema",
        "json_schema": {
            "name": "empty_schema",
            "schema": {"type": "object", "properties": {}, "additionalProperties": True},
        },
    },
)
```

## Precontext metadata

`precontext` on the response (under `response.precontext` for the OpenAI SDK / Python, or `response.body?.precontext` for the Vercel AI SDK) contains:

- `name: "stt"` — raw transcript with `chunks[].timestamp` arrays.
- `name: "translate"` — when translation is requested, includes `source_language`, `target_language`, `translated_text`.

## Plain transcription

Use `generateText` (Vercel AI SDK) or omit `response_format` (OpenAI / LangChain) when you only need raw text.

## Tips

- Use canonical MIME types: `audio/mpeg` for MP3, `audio/wav` for WAV, `audio/mp4` for M4A.
- Include "identify the speakers" in the prompt to trigger diarization.
- Specify the target language in the prompt for translation, e.g. "translate to Chinese".
- Use `<task>speech_to_text</task>` for long audio or word-level timestamps.
