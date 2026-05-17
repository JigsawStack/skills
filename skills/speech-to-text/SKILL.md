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

Transcribe audio files into text with speaker detection, timestamps, and language identification using Interfaze AI.

## When to use this skill

- The user has an audio file and wants it transcribed to text
- The input is a voice note, meeting recording, podcast, interview, or phone call
- The user wants speaker identification or diarization from an audio source
- The user asks for timestamps alongside the transcript
- The user says things like "transcribe this", "what does this audio say", "convert this recording to text"
- The user wants to detect the language spoken in an audio file

## When not to use this skill

- The input is an image or screenshot — use the `ocr` skill instead
- The user wants to generate speech from text (text-to-speech) — not supported
- The input is already text that needs reformatting — no transcription needed
- The user wants to extract data from a video's visual content — use `ocr` or `object-detection`

## Workflow

1. Confirm the input is an audio file URL or file reference.
2. Determine what the user needs: plain transcript, speaker-labeled transcript, timestamps, or language detection.
3. Build a Zod schema matching the desired output (text only, or with speakers and timestamps).
4. Use `generateObject` with the audio passed in the message content array.
5. Return the transcript in the requested format.

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

## Example: Basic transcription

```ts
const response = await generateObject({
  model: interfaze.chat("interfaze-beta"),
  messages: [
    {
      role: "user",
      content: [
        { type: "text", text: "Transcribe the audio file" },
        { type: "file", data: "https://example.com/voice-note.mp3", mediaType: "audio/mpeg" },
      ],
    },
  ],
  schema: z.object({
    text: z.string(),
  }),
});
```

## Example: Transcribe with speaker diarization

```ts
const response = await generateObject({
  model: interfaze.chat("interfaze-beta"),
  messages: [
    {
      role: "user",
      content: [
        { type: "text", text: "Transcribe and identify the speakers in the audio file" },
        { type: "file", data: "https://example.com/meeting.mp3", mediaType: "audio/mpeg" },
      ],
    },
  ],
  schema: z.object({
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
  }),
});
```

## Example: Audio translation

```ts
const response = await generateObject({
  model: interfaze.chat("interfaze-beta"),
  prompt:
    "Transcribe the audio file and translate it to Chinese https://example.com/voice-note.mp3",
  schema: z.object({
    translated_text: z.string().describe("translated text"),
    original_language_code: z.string(),
    translated_language_code: z.string(),
  }),
});
```

## Faster, cheaper raw transcription

For long audio (1hr+) or to minimize cost and latency, run as a single task with `<task>speech_to_text</task>` in the system message. The model returns a fixed structure (text plus `chunks` with `timestamp: [start, end]`) without activating the rest of the model.

```ts
const response = await generateObject({
  model: interfaze.chat("interfaze-beta"),
  system: "<task>speech_to_text</task>",
  schema: z.any(),
  messages: [
    {
      role: "user",
      content: [
        {
          type: "text",
          text: "Transcribe the audio file https://example.com/long-audio.mp3",
        },
      ],
    },
  ],
});
```

Passing the URL inline in the prompt (instead of using a `file` part) is supported and slightly faster.

## Available references

- [references/api.md](references/api.md) — API usage details for STT
- [references/examples.md](references/examples.md) — Additional trigger examples and patterns
