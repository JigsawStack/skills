---
name: speech-to-text
description: >
  Use Interfaze AI for speech-to-text transcription from audio files, voice notes,
  recordings, podcasts, and meeting audio. Activate when the user wants to transcribe
  audio to text, identify speakers, detect language, or extract timestamps — even if
  they say "what did they say in this recording" or "convert this audio to text"
  without explicitly mentioning transcription.
---

# STT — Speech-to-Text with Interfaze AI

Transcribe audio files into text with speaker diarization, timestamps, translation, and audio analysis using Interfaze AI.

## When to use this skill

- The user has an audio file and wants it transcribed to text
- The input is a voice note, meeting recording, podcast, interview, or phone call
- The user wants speaker identification or diarization from an audio source
- The user asks for timestamps alongside the transcript
- The user says things like "transcribe this", "what does this audio say", "convert this recording to text"
- The user wants to translate audio between languages

## When not to use this skill

- The input is an image or screenshot — use the `ocr` skill instead
- The user wants to generate speech from text (text-to-speech) — not supported
- The input is already text that needs reformatting — no transcription needed
- The user wants to extract data from a video's visual content — use `ocr` or `object-detection`

## Workflow

1. Confirm the input is an audio file URL or file reference.
2. Determine what the user needs: plain transcript, speaker-labeled transcript, timestamps, or translation.
3. Build a schema matching the desired output (Zod for TypeScript, Pydantic for Python).
4. Pass the audio in the message content array. URLs can also be passed inline in the prompt text.

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

## Example: Basic transcription

### TypeScript — OpenAI SDK

```ts
import { z } from "zod";
import { zodResponseFormat } from "openai/helpers/zod";

const STTSchema = z.object({
  text: z.string(),
});

const response = await interfaze.chat.completions.create({
  model: "interfaze-beta",
  messages: [
    {
      role: "user",
      content: [
        { type: "text", text: "Transcribe the audio file" },
        {
          type: "file",
          file: {
            filename: "voice-note.mp3",
            file_data: "https://example.com/voice-note.mp3",
          },
        },
      ],
    },
  ],
  response_format: zodResponseFormat(STTSchema, "stt_schema"),
});

console.log(response.choices[0].message.content);
```

### TypeScript — Vercel AI SDK

```ts
import { generateObject } from "ai";
import { z } from "zod";

const STTSchema = z.object({
  text: z.string(),
});

const { object } = await generateObject({
  model: interfaze.chat("interfaze-beta"),
  schema: STTSchema,
  messages: [
    {
      role: "user",
      content: [
        { type: "text", text: "Transcribe the audio file" },
        {
          type: "file",
          data: "https://example.com/voice-note.mp3",
          mediaType: "audio/mpeg",
        },
      ],
    },
  ],
});

console.log(object);
```

### TypeScript — LangChain SDK

```ts
import { z } from "zod";

const STTSchema = z.object({
  text: z.string(),
});

const structuredModel = interfaze.withStructuredOutput(STTSchema);

const response = await structuredModel.invoke([
  {
    role: "user",
    content: [
      { type: "text", text: "Transcribe the audio file" },
      {
        type: "file",
        file: {
          filename: "voice-note.mp3",
          file_data: "https://example.com/voice-note.mp3",
        },
      },
    ],
  },
]);

console.log(response);
```

### Python — OpenAI SDK

```python
from pydantic import BaseModel

class STTSchema(BaseModel):
    text: str

response = interfaze.chat.completions.create(
    model="interfaze-beta",
    messages=[
        {
            "role": "user",
            "content": [
                {"type": "text", "text": "Transcribe the audio file"},
                {
                    "type": "file",
                    "file": {
                        "filename": "voice-note.mp3",
                        "file_data": "https://example.com/voice-note.mp3",
                    },
                },
            ],
        }
    ],
    response_format={
        "type": "json_schema",
        "json_schema": {"name": "stt_schema", "schema": STTSchema.model_json_schema()},
    },
)

print(response.choices[0].message.content)
```

### Python — LangChain SDK

```python
from langchain_core.messages import HumanMessage
from pydantic import BaseModel

class STTSchema(BaseModel):
    text: str

structured_llm = interfaze.with_structured_output(STTSchema)

response = structured_llm.invoke([
    HumanMessage(content=[
        {"type": "text", "text": "Transcribe the audio file"},
        {
            "type": "file",
            "file": {
                "filename": "voice-note.mp3",
                "file_data": "https://example.com/voice-note.mp3",
            },
        },
    ])
])

print(response)
```

## Example: Speaker diarization

Use a `chunks` array of `{ speaker_id, text, start_time, end_time }`. `speaker_id` is a string. Up to 50 speakers are supported.

### TypeScript — OpenAI SDK

```ts
import { z } from "zod";
import { zodResponseFormat } from "openai/helpers/zod";

const DiarizationSchema = z.object({
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
});

const response = await interfaze.chat.completions.create({
  model: "interfaze-beta",
  messages: [
    {
      role: "user",
      content: [
        { type: "text", text: "Transcribe and identify the speakers in the audio file" },
        {
          type: "file",
          file: {
            filename: "meeting.mp3",
            file_data: "https://example.com/meeting.mp3",
          },
        },
      ],
    },
  ],
  response_format: zodResponseFormat(DiarizationSchema, "diarization_schema"),
});
```

### TypeScript — Vercel AI SDK

```ts
import { generateObject } from "ai";
import { z } from "zod";

const DiarizationSchema = z.object({
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
});

const { object } = await generateObject({
  model: interfaze.chat("interfaze-beta"),
  schema: DiarizationSchema,
  messages: [
    {
      role: "user",
      content: [
        { type: "text", text: "Transcribe and identify the speakers in the audio file" },
        { type: "file", data: "https://example.com/meeting.mp3", mediaType: "audio/mpeg" },
      ],
    },
  ],
});
```

### Python — OpenAI SDK

```python
from typing import List
from pydantic import BaseModel

class Chunk(BaseModel):
    speaker_id: str
    text: str
    start_time: float
    end_time: float

class DiarizationSchema(BaseModel):
    full_text: str
    chunks: List[Chunk]
    number_of_speakers: int

response = interfaze.chat.completions.create(
    model="interfaze-beta",
    messages=[
        {
            "role": "user",
            "content": [
                {"type": "text", "text": "Transcribe and identify the speakers in the audio file"},
                {
                    "type": "file",
                    "file": {
                        "filename": "meeting.mp3",
                        "file_data": "https://example.com/meeting.mp3",
                    },
                },
            ],
        }
    ],
    response_format={
        "type": "json_schema",
        "json_schema": {
            "name": "diarization_schema",
            "schema": DiarizationSchema.model_json_schema(),
        },
    },
)
```

The LangChain and Python LangChain variants follow the basic-transcription pattern with `DiarizationSchema` swapped in.

## Example: Audio translation

### TypeScript — Vercel AI SDK

```ts
import { generateObject } from "ai";
import { z } from "zod";

const TranslationSchema = z.object({
  translated_text: z.string().describe("translated text"),
  original_language_code: z.string(),
  translated_language_code: z.string(),
});

const { object } = await generateObject({
  model: interfaze.chat("interfaze-beta"),
  schema: TranslationSchema,
  prompt:
    "Transcribe the audio file and translate it to Chinese https://example.com/voice-note.mp3",
});
```

### Python — OpenAI SDK

```python
from pydantic import BaseModel, Field

class TranslationSchema(BaseModel):
    translated_text: str = Field(..., description="translated text")
    original_language_code: str
    translated_language_code: str

response = interfaze.chat.completions.create(
    model="interfaze-beta",
    messages=[
        {
            "role": "user",
            "content": "Transcribe the audio file and translate it to Chinese https://example.com/voice-note.mp3",
        }
    ],
    response_format={
        "type": "json_schema",
        "json_schema": {"name": "translation_schema", "schema": TranslationSchema.model_json_schema()},
    },
)
```

The other SDK variants follow the same pattern as the basic transcription example.

## Faster, cheaper raw transcription (long audio)

For long audio (1hr+) or to minimize cost and latency, run as a single task with `<task>speech_to_text</task>` in the system message. The model returns a fixed structure with `text` plus `chunks: [{ timestamp: [start, end], text }]`. See [references/api.md](references/api.md) for the full multi-SDK breakdown.

## Available references

- [references/api.md](references/api.md) — API usage details, raw task mode, audio MIME types
- [references/examples.md](references/examples.md) — Additional trigger examples and patterns
