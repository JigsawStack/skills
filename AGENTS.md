# Interfaze AI — Agent Guidelines

This repo contains installable Agent Skills for Interfaze AI. Follow these rules when working with or contributing to these skills.

## General rules

- Interfaze AI is a proprietary model. Which easily work with OpenAI SDK by setting the baseURL to https://interfaze.ai/v1
- Only document how to use the API — inputs, outputs, and patterns.
- Interfaze exposes an OpenAI-compatible API. Use the Vercel AI SDK (`@ai-sdk/openai`) with `baseURL` pointed at the Interfaze endpoint.
- The model name is `interfaze-beta`.
- Always require `INTERFAZE_API_KEY` as an environment variable.

## Skill authoring

- Keep skill descriptions precise and trigger-oriented. The `description` field is the primary routing surface.
- Prefer narrow, capability-specific skills over broad catch-all skills.
- Keep examples concrete and realistic. Show actual code with Zod schemas and Vercel AI SDK calls.
- Put reusable API details and examples in `references/` files, not inside `SKILL.md`.
- Do not duplicate long API documentation inside `SKILL.md`. Reference it instead.
- Maintain naming consistency: folder names must match the `name` field in frontmatter.

## Writing style

- Use direct, practical language.
- Avoid marketing language, hype, or superlatives.
- Lead with what the user needs to do, not what the product can do.
- Prefer imperative instructions: "Use this when..." not "This skill provides..."

## API conventions

- Use `generateObject` from the Vercel AI SDK when structured output is needed.
- Use `generateText` when plain text output is sufficient.
- Define schemas with Zod. Use `.describe()` on fields when the field name alone is ambiguous.
- Pass images as `{ type: "image", image: "<url>" }` in message content arrays.
- Pass audio as `{ type: "file", data: "<url>", mediaType: "audio/mp3" }` in message content arrays.

## Source of truth

- The Interfaze API behavior is the source of truth. If a skill contradicts observed API behavior, update the skill.
- Do not invent capabilities that have not been tested against the live API.
