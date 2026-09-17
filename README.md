# AI Content Pipeline for n8n

A three-stage content automation pipeline built in n8n for collecting, transforming, reviewing, and publishing social-media posts.

## Overview

The project separates the content lifecycle into three independent workflows:

1. **Content generation** — takes pre-collected posts from Supabase, filters eligible records, rewrites them for the target channel, generates visual content, and stores prepared posts in Airtable.
2. **AI quality control** — validates prepared posts before publication, checks formatting and text quality, and writes an approval decision plus review comments back to Airtable.
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
   |  validation / moderation / approval
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
- **Cloudinary** — media storage
- **Telegram Bot API** — Telegram publishing
- **MAX API** — MAX publishing

## Content formats

The publishing layer supports:

- single image + text;
- video + text;
- multi-image carousel (up to six images).

## Repository structure

```text
workflows/
  01_content_generation.json
  02_ai_quality_control.json
  03_publishing.json
README.md
```

## Security

Production credentials are not stored in this repository. After importing the workflows into n8n, configure your own credentials for Supabase, Airtable, OpenAI/LLM providers, Cloudinary, Telegram, and MAX.

Instance-specific credential references, webhook identifiers, private chat IDs, and Airtable resource identifiers are removed or replaced with placeholders in the public workflow exports.

## Status

The project is being published incrementally. Workflow exports and documentation will be added in separate commits to keep the repository history aligned with the actual system architecture.
