# STT — Trigger Examples

## User asks that should activate this skill

- "Transcribe this audio file"
- "What do they say in this recording?"
- "Convert this voice note to text"
- "Who are the speakers in this meeting recording?"
- "Get me a transcript with timestamps from this podcast"
- "What language is spoken in this audio?"
- "Transcribe this MP3 and identify each speaker"
- "Turn this interview recording into text"

## User asks that should NOT activate this skill

- "Read the text in this image" — use `ocr`
- "Extract data from this PDF scan" — use `ocr`
- "Generate speech from this text" — not supported
- "Summarize this article" — no audio input involved
- "Detect objects in this photo" — use `object-detection`

## Ideal agent behavior

1. Accept the audio file (URL or file reference).
2. Determine whether the user wants plain text, speaker labels, timestamps, or language detection.
3. Build a Zod schema if structured output is needed.
4. Call `generateObject` or `generateText` with the audio in the message content.
5. Return the transcript in the requested format.
