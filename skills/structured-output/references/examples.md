# Structured Output — Trigger Examples

## User asks that should activate this skill

- "Give me this as JSON"
- "Parse this text into structured fields"
- "Extract the name, email, and phone from this id"
- "Return a typed object with the results for this audio"
- "Convert this data into a schema-validated response"
- "I need the output as an array of objects with these fields"
- "Normalize this messy data into a clean JSON structure"

## User asks that should NOT activate this skill

- "Summarize this article" — plain text output, no schema needed
- "Explain how this works" — conversational response
- "Transcribe this audio" — use `speech-to-text`

## Ideal agent behavior

1. Identify what fields and structure the user expects.
2. Define a Zod schema matching that structure.
3. Call `generateObject` with the schema.
4. Return `response.object` directly — it is already typed and validated.
