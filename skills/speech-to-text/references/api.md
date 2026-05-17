# Speech to Text — API Reference

## Endpoint

Interfaze AI uses an OpenAI-compatible chat completions endpoint:

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

## Audio input format

Audio is passed in the message content array:

```ts
{
  role: "user",
  content: [
    { type: "file", data: "<audio-url>", mediaType: "audio/mpeg" },
    { type: "text", text: "Transcribe this audio." },
  ],
}
```

Supported audio formats:
- MP3, M4A, WAV, and other common audio formats
- Public URLs (HTTPS)

## Structured transcription

Use `generateObject` with a Zod schema for typed transcript output:

```ts
const response = await generateObject({
  model: interfaze.chat("interfaze-beta"),
  messages: [...],
  schema: z.object({
    text: z.string(),
  }),
});
```

## Speaker diarization

For speaker-labeled output, ask the model to identify speakers and use a `chunks` array. `speaker_id` is a string. Up to 50 speakers are supported.

```ts
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
})
```

You can extend each chunk with additional analysis fields, e.g. `sentiment: z.enum(["positive", "negative", "neutral"])`.

## Run task mode (raw output)

For long audio (1hr+) or maximum speed and lowest cost, set `<task>speech_to_text</task>` in the system message. The response contains a fixed structure with `text` and `chunks: [{ timestamp: [start, end], text }]`. Pass the audio URL inline in the prompt:

```ts
const response = await generateObject({
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

## Plain transcription

Use `generateText` when you only need the raw transcript:

```ts
const response = await generateText({
  model: interfaze.chat("interfaze-beta"),
  messages: [...],
});

// response.text contains the transcript
```

## Tips

- Use the canonical MIME type: `audio/mpeg` for MP3, `audio/wav` for WAV, `audio/mp4` for M4A.
- Include "identify the speakers" in the prompt to trigger diarization.
- Specify the target language in the prompt for translation, e.g. "translate to Chinese".
- Use `<task>speech_to_text</task>` for long audio or word-level timestamps.
