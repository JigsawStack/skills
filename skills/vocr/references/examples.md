# VOCR — Trigger Examples

## User asks that should activate this skill

- "Extract the text from this screenshot"
- "Read the data on this invoice image"
- "What does this receipt say?"
- "Pull the name and address from this scanned ID"
- "OCR this PDF scan"
- "Get the line items and totals from this photo of a receipt"
- "Extract all the fields from this form image"
- "Read the menu items and prices from this photo"
- "What text is in this image?"
- "Translate the text in this image to Hindi" (OCR + translation)

## User asks that should NOT activate this skill

- "Transcribe this audio recording" — use `speech-to-text`
- "Convert this JSON to a different schema" — use `structured-output`
- "Find all the cars in this photo" — use `object-detection`
- "Summarize this text file" — no vision needed
- "Scrape the products from this URL" — use `web-scraping`

## Ideal agent behavior

1. Accept the image input (URL or file path).
2. Determine whether the user wants raw text or structured fields.
3. Build a Zod schema if structured extraction is needed.
4. Call `generateObject` or `generateText` with the image and prompt.
5. Return the result in the format the user requested.
