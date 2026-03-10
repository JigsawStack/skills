# Speech to Text — API Reference

## Endpoint

Interfaze AI uses an OpenAI-compatible chat completions endpoint:

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

## Audio input format

Audio is passed in the message content array:

```ts
{
  role: "user",
  content: [
    { type: "file", data: "<audio-url>", mediaType: "audio/mp3" },
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
    language: z.string(),
    speakers: z.array(z.object({
      speaker_id: z.number(),
      text: z.string(),
      start_time: z.number().describe("start time in seconds"),
      end_time: z.number().describe("end time in seconds"),
    })),
  }),
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

- Include "identify speakers" in the prompt to get speaker diarization.
- Add timestamp fields to the schema when the user needs time references.
- Specify the expected language in the prompt for better accuracy on multilingual audio.
