# Interfaze AI — Agent Guidelines

This repo contains installable Agent Skills for Interfaze AI. Follow these rules when working with or contributing to these skills.

## General rules

- Interfaze AI is a proprietary multimodal model accessible through an OpenAI-compatible API.
- API base URL: `https://api.interfaze.ai/v1`. Model name: `interfaze-beta`.
- Always require `INTERFAZE_API_KEY` as an environment variable.
- Only document how to use the API — inputs, outputs, and patterns.

## Supported SDKs and languages

Every skill must show code examples for all supported SDK + language combinations the docs cover:

- **TypeScript**: OpenAI SDK (`openai`), Vercel AI SDK (`@ai-sdk/openai` + `ai`), LangChain SDK (`@langchain/openai`).
- **Python**: OpenAI SDK (`openai`), LangChain SDK (`langchain-openai`).

Use these section labels consistently inside each example, in this order:

1. `### TypeScript — OpenAI SDK`
2. `### TypeScript — Vercel AI SDK`
3. `### TypeScript — LangChain SDK`
4. `### Python — OpenAI SDK`
5. `### Python — LangChain SDK`

## Skill authoring

- Keep skill descriptions precise and trigger-oriented. The `description` field is the primary routing surface.
- Prefer narrow, capability-specific skills over broad catch-all skills.
- Put reusable API details and long examples in `references/` files, not inside `SKILL.md`.
- Maintain naming consistency: folder names must match the `name` field in frontmatter.

## Writing style

- Use direct, practical language.
- Avoid marketing language, hype, or superlatives.
- Lead with what the user needs to do, not what the product can do.
- Prefer imperative instructions: "Use this when..." not "This skill provides..."

## API conventions

### Vercel AI SDK (TypeScript)

- Use `generateObject` when structured output is needed; use `generateText` for plain text.
- Define schemas with Zod. Use `.describe()` on fields when the field name alone is ambiguous.
- Pass images as `{ type: "image", image: "<url>" }` in message content arrays. Add `mediaType` (e.g. `"image/jpeg"`, `"image/png"`) when the URL extension is ambiguous.
- Pass audio/PDF as `{ type: "file", data: "<url>", mediaType: "<mime>" }`. Use canonical MIME types: `audio/mpeg` for MP3, `audio/wav`, `audio/mp4` for M4A, `application/pdf` for PDF.

### OpenAI SDK (TypeScript and Python)

- Use `interfaze.chat.completions.create({ model: "interfaze-beta", ... })`.
- For structured output use `response_format`:
  - TypeScript: `zodResponseFormat(Schema, "schema_name")` from `openai/helpers/zod`.
  - Python: `{ "type": "json_schema", "json_schema": { "name": "schema_name", "schema": Model.model_json_schema() } }` with a `pydantic.BaseModel`.
- Pass images as `{ type: "image_url", image_url: { url: "<url>" } }`.
- Pass audio/PDF as `{ type: "file", file: { filename: "<name>", file_data: "<url-or-base64>" } }`.

### LangChain SDK (TypeScript and Python)

- Use `interfaze.withStructuredOutput(Schema)` / `interfaze.with_structured_output(Model)` for structured output.
- Use `interfaze.invoke([...])` for plain text.
- Pass images and files the same way as the OpenAI SDK (`image_url` and `file`).

## Source of truth

- The Interfaze API behavior is the source of truth. If a skill contradicts observed API behavior, update the skill.
- Do not invent capabilities that have not been tested against the live API.
