# Interfaze AI — Agent Skills

Installable [Agent Skills](https://agentskills.io) for [Interfaze AI](https://interfaze.ai). This repo follows the open Agent Skills format and is compatible with the `skills` CLI.

## What is Interfaze AI?

Interfaze AI is a proprietary multimodal AI model accessible through an OpenAI-compatible API. It handles vision OCR, speech-to-text, structured output, object detection, web scraping, and document processing through a single unified endpoint — no tool routing or multi-provider setup required.

## Included skills

| Skill                                          | Purpose                                                                                |
| ---------------------------------------------- | -------------------------------------------------------------------------------------- |
| [ocr](skills/ocr/)                             | OCR and visual text extraction from images, scans, PDFs, invoices, receipts, and forms |
| [speech-to-text](skills/speech-to-text/)       | Speech-to-text transcription from audio files, voice notes, and recordings             |
| [structured-output](skills/structured-output/) | Convert model responses into strict JSON or schema-constrained structured objects      |
| [object-detection](skills/object-detection/)   | Detect and locate objects in images with bounding box coordinates                      |
| [web-search](skills/web-search/)               | Search the web for current information, facts, prices, and news                        |
| [web-scraping](skills/web-scraping/)           | Extract structured data from specific web pages and URLs                               |

## Installation

Install all skills:

```bash
npx skills add JigsawStack/interfaze-skills
```

Install a single skill:

```bash
npx skills add JigsawStack/interfaze-skills --skill ocr
```

List available skills without installing:

```bash
npx skills add JigsawStack/interfaze-skills --list
```

## Setup

Interfaze AI uses an OpenAI-compatible API. Set your API key and base URL:

```ts
import { createOpenAI } from "@ai-sdk/openai";

const interfaze = createOpenAI({
  baseURL: "https://api.interfaze.ai/v1",
  apiKey: process.env.INTERFAZE_API_KEY,
});
```

Use the model name `interfaze-beta` with Vercel AI SDK functions like `generateObject` and `generateText`.

## When to use AGENTS.md vs SKILL.md

- **AGENTS.md** contains repo-wide rules and conventions that apply to all skills. Agents load this as passive context.
- **SKILL.md** files contain specialized instructions for a single capability. Agents load these on-demand when a task matches the skill description.

## Format

This repo follows the [Agent Skills open specification](https://agentskills.io/specification). Skills work with 18+ AI agents including Claude Code, GitHub Copilot, Cursor, Cline, and others.

## License

MIT — see [LICENSE](LICENSE).
