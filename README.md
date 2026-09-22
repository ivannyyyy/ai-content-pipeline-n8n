# AI Content Pipeline for n8n

A three-stage AI content automation system built in n8n for generating, reviewing, and publishing social-media content.

## Overview

The project separates the content lifecycle into three connected workflows:

1. **Content Generation** — takes source records from Supabase, rewrites content with LLMs, generates visual assets, stores media, and creates prepared posts in Airtable.
2. **AI Quality Control** — applies deterministic validation and AI moderation to text and images before publication.
3. **Publishing** — selects approved Airtable records and publishes them to Telegram and MAX, then records delivery state back in Airtable.

## Architecture

~~~text
Supabase
   |
   v
[01 Content Generation]
   |  LLM rewrite / media generation
   v
Airtable (content plan)
   |
   v
[02 AI Quality Control]
   |  deterministic checks / AI moderation
   v
Airtable (approved content)
   |
   v
[03 Publishing]
   +------> Telegram
   |
   +------> MAX
~~~

## Main components

- **n8n** — workflow orchestration
- **Supabase** — source content storage
- **Airtable** — content plan and workflow state
- **OpenAI / LLM nodes** — text generation and moderation
- **AI image analysis** — visual quality control
- **Image generation / media processing** — content assets
- **Cloudinary** — media storage
- **Telegram Bot API** — Telegram publishing
- **MAX API** — MAX publishing

## Content formats

The pipeline supports:

- single image + text;
- video + text;
- multi-image carousel / media group with up to six images.

## Published workflows

### 01 — Content Generation

Workflow:

[workflows/01_content_generation.json](workflows/01_content_generation.json)

Documentation:

[docs/content-generation.md](docs/content-generation.md)

Configuration checklist:

[config/content-generation.example.json](config/content-generation.example.json)

This stage handles source selection, LLM transformation, media generation, Cloudinary upload, Airtable preparation, source-state updates, error notifications, and handoff to quality control.

### 02 — AI Quality Control

Workflow:

[workflows/02_ai_quality_control.json](workflows/02_ai_quality_control.json)

Documentation:

[docs/ai-quality-control.md](docs/ai-quality-control.md)

This stage acts as a pre-publication quality gate. Deterministic checks run first, followed by AI review of text and, for multi-image content, the generated visuals. The result is written back to Airtable.

### 03 — Publishing

Workflow:

[workflows/03_publishing.json](workflows/03_publishing.json)

Documentation:

[docs/publishing.md](docs/publishing.md)

This stage selects approved content, routes it by media type, publishes to Telegram and MAX, and records platform-specific publication timestamps back in Airtable.

## End-to-end state flow

~~~text
source record
   |
   v
generated content
   |
   v
AI-reviewed content
   |
   +--> rejected ----> review / correction
   |
   v
approved
   |
   +--> Telegram ----> tg timestamp
   |
   +--> MAX ---------> max timestamp
~~~

The pipeline keeps deterministic rules, AI decisions, content state, and delivery state explicit instead of hiding them inside a single monolithic agent.

## Quick setup

1. Import the three JSON workflows into n8n.
2. Configure your own credentials for Supabase, Airtable, OpenAI/LLM providers, Cloudinary, Telegram, and MAX.
3. Replace every **YOUR_...** placeholder.
4. Review project-specific table names, channel values, prompts, schedules, and caption templates.
5. Connect the Content Generation workflow to the AI Quality Control workflow.
6. Test each stage independently.
7. Test the complete generation → review → publishing flow.
8. Enable production schedules only after all state transitions work correctly.

## Repository structure

~~~text
workflows/
  01_content_generation.json
  02_ai_quality_control.json
  03_publishing.json

docs/
  content-generation.md
  ai-quality-control.md
  publishing.md

config/
  content-generation.example.json

README.md
~~~

## Security

Production credentials are not stored in this repository.

The public workflow exports remove or replace deployment-specific credential references, private chat identifiers, workflow identifiers, webhook identifiers, and Airtable resource identifiers where required.

After import, configure your own credentials and review all project-specific destinations before enabling any workflow.

## Project status

**Complete public portfolio version.**

The repository now contains the full three-stage automation architecture:

**Content Generation → AI Quality Control → Publishing**

The original production system was built as a working content pipeline; this repository is a sanitized public version intended to demonstrate workflow architecture, integrations, state management, AI-assisted quality control, and multi-platform publishing.
