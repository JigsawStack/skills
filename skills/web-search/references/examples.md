# Web Search — Trigger Examples

## User asks that should activate this skill

- "What's the current price of Bitcoin?"
- "Look up the latest news about SpaceX"
- "Find me information about this company"
- "What are the top restaurants in San Francisco?"
- "Search for the release date of the new iPhone"
- "Who won the game last night?"
- "Find the documentation for this library"

## User asks that should NOT activate this skill

- "Scrape all products from this Amazon page" — use `web-scraping`
- "Extract text from this image" — use `vocr`
- "Transcribe this recording" — use `speech-to-text`
- "What is 2 + 2?" — no web search needed
- "Explain the difference between let and const" — general knowledge

## Ideal agent behavior

1. Determine what current information the user needs.
2. Formulate a clear, specific search prompt.
3. Choose `generateObject` (structured) or `generateText` (plain) based on the use case.
4. Return results in the requested format.
