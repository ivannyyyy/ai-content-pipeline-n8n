# AI Content Pipeline for n8n

A three-stage content automation pipeline built in n8n for collecting, transforming, reviewing, and publishing social-media posts.

## Overview

The project separates the content lifecycle into three independent workflows:

1. **Content generation** — takes pre-collected posts from Supabase, filters eligible records, rewrites them for the target channel, generates visual content, and stores prepared posts in Airtable.
2. **AI quality control** — validates prepared posts before publication, checks media, formatting and text quality, and writes an approval decision plus review comments back to Airtable.
3. **Publishing** — selects approved posts according to the publication schedule and publishes them to Telegram and MAX, supporting single-image, video, and multi-image posts.

## Architecture

```text
Supabase
   |
   v
[01 Content Generation]
   |  rewrite / relevance / visual generation
   v
Airtable (content plan)
   |
   v
[02 AI Quality Control]
   |  deterministic checks / AI moderation / approval
   v
Airtable (approved content)
   |
   v
[03 Publishing]
   +------> Telegram
   |
   +------> MAX
```

## Main components

- **n8n** — workflow orchestration
- **Supabase** — source content storage
- **Airtable** — content plan and workflow state
- **LLM agents** — rewriting and quality control
- **Image generation / media processing** — visual assets for posts
- **AI image analysis** — visual quality control for multi-image content
- **Cloudinary** — media storage
- **Telegram Bot API** — Telegram publishing
- **MAX API** — MAX publishing

## Content formats

The pipeline is designed for:

- single image + text;
- video + text;
- multi-image carousel (up to six images).

## Published workflows

### 01 — Content Generation

Sanitized workflow:

[`workflows/01_content_generation.json`](workflows/01_content_generation.json)

It contains source selection, LLM transformation, media generation, Cloudinary upload, Airtable preparation, source-state updates, operational alerting, and handoff to quality control.

Documentation:

[`docs/content-generation.md`](docs/content-generation.md)

Configuration checklist:

[`config/content-generation.example.json`](config/content-generation.example.json)

### 02 — AI Quality Control

Sanitized workflow:

[`workflows/02_ai_quality_control.json`](workflows/02_ai_quality_control.json)

It acts as a pre-publication quality gate: deterministic media/text checks run first, then AI moderation reviews text and, for multi-image content, the generated visuals. Approval state and review comments are written back to Airtable.

Documentation:

[`docs/ai-quality-control.md`](docs/ai-quality-control.md)

## Repository structure

```text
workflows/
  01_content_generation.json
  02_ai_quality_control.json

docs/
  content-generation.md
  ai-quality-control.md

config/
  content-generation.example.json

README.md
```

Planned incremental addition:

```text
workflows/03_publishing.json
```

## Security

Production credentials are not stored in this repository. After importing the workflows into n8n, configure your own credentials for Supabase, Airtable, OpenAI/LLM providers, Cloudinary, Telegram, and MAX.

Instance-specific credential references, workflow identifiers, private chat IDs, webhook identifiers, and Airtable resource identifiers are removed or replaced with placeholders in the public workflow exports.

## Status

The project is being published incrementally in small logical commits so that the repository history reflects the actual system architecture.

Current public stage: **Content Generation + AI Quality Control, with setup documentation**.
