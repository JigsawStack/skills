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

- The input is an image or screenshot — use the `vocr` skill instead
- The user wants to generate speech from text (text-to-speech) — not supported
- The input is already text that needs reformatting — no transcription needed
- The user wants to extract data from a video's visual content — use `vocr` or `object-detection`

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
  baseURL: "https://interfaze.ai/v1",
  apiKey: process.env.INTERFAZE_API_KEY,
});
```

## Example: Transcribe with speaker detection

```ts
const response = await generateObject({
  model: interfaze.chat("interfaze-beta"),
  messages: [
    {
      role: "user",
      content: [
        { type: "file", data: "https://example.com/meeting.mp3", mediaType: "audio/mp3" },
        { type: "text", text: "Transcribe this audio file and identify speakers." },
      ],
    },
  ],
  schema: z.object({
    text: z.string(),
    speakers: z.array(
      z.object({
        speaker_id: z.number(),
        text: z.string(),
        start_time: z.number().describe("start time in seconds"),
        end_time: z.number().describe("end time in seconds"),
      })
    ),
    language: z.string(),
  }),
});
```

## Example: Simple transcript

```ts
const response = await generateText({
  model: interfaze.chat("interfaze-beta"),
  messages: [
    {
      role: "user",
      content: [
        { type: "file", data: "https://example.com/voice-note.mp3", mediaType: "audio/mp3" },
        { type: "text", text: "Transcribe this audio." },
      ],
    },
  ],
});
```

## Available references

- [references/api.md](references/api.md) — API usage details for STT
- [references/examples.md](references/examples.md) — Additional trigger examples and patterns
