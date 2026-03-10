# Web Scraping — Trigger Examples

## User asks that should activate this skill

- "Scrape all products from this Amazon page"
- "Get the listings and prices from this URL"
- "Extract the profile data from this LinkedIn page"
- "Pull all the job postings from this careers page"
- "Get the article titles and links from Hacker News"
- "Extract the menu items and prices from this restaurant's website"
- "Scrape the reviews from this product page"
- "Get all the data from this table on this webpage"

## User asks that should NOT activate this skill

- "Search for the best restaurants in NYC" — use `web-search` (no specific URL)
- "What's the current price of Bitcoin?" — use `web-search`
- "Read the text in this image" — use `vocr`
- "Transcribe this audio" — use `speech-to-text`
- "Convert this JSON to a different format" — use `structured-output`

## Ideal agent behavior

1. Confirm the target URL.
2. Determine what data the user wants from the page.
3. Build a Zod schema matching the desired output.
4. Call `generateObject` with the URL in the prompt and the schema.
5. Return the extracted data.
