# Content Generation Workflow

This document describes the public, sanitized export in `workflows/01_content_generation.json`.

## Purpose

The workflow prepares social-media content before quality control and publication. It pulls source records from Supabase, transforms the text with an LLM, generates media, stores the prepared post in Airtable, updates source state, and then hands the result to the separate AI quality-control workflow.

## High-level flow

```text
Supabase
   |
   v
Select eligible source post
   |
   +-----------------------------+
   |                             |
   v                             v
Single-image path           Multi-image path
   |                             |
LLM rewrite                  LLM/content planning
   |                             |
Image prompt                 Generate up to 6 images
   |                             |
Image generation             Upload assets
   |                             |
Cloudinary                   Airtable content record
   |                             |
Airtable content record      Final text generation
   |                             |
Mark source as done          Mark source as done
   |                             |
   +--------------+--------------+
                  |
                  v
        AI Quality Control
```

## What the exported workflow contains

The current export contains 61 n8n nodes and two main generation paths:

### 1. Single-image post

The workflow selects one eligible Supabase record, generates an image prompt, creates the visual, rewrites the post text, uploads the media to Cloudinary, and creates a prepared Airtable record.

In the exported version, the source selection checks records with:

- `status = new`
- `relevant = true`

After a successful preparation step, the source record is marked as `done`.

### 2. Multi-image post

The second path prepares a multi-image post with up to six generated assets. The assets are uploaded individually and written into separate Airtable attachment fields before the final post text is generated.

The source record is then marked as processed and the prepared content is passed to quality control.

## External services

The workflow uses:

- **Supabase** — source post storage and processing state;
- **OpenAI / LLM nodes** — text transformation, planning, and prompt generation;
- **image-generation HTTP/API calls** — media generation;
- **Cloudinary** — generated asset storage;
- **Airtable** — prepared content plan and workflow state;
- **Telegram** — operational error notifications;
- **n8n Execute Workflow** — handoff to the AI quality-control workflow.

## Required configuration

Before using the workflow, replace the public placeholders and bind your own n8n credentials.

Placeholders present in the public export:

| Placeholder | Purpose |
|---|---|
| `YOUR_AIRTABLE_BASE_ID` | Airtable content-plan base |
| `YOUR_AIRTABLE_TABLE_ID` | Airtable table used for prepared posts |
| `YOUR_QA_WORKFLOW_ID` | n8n workflow ID for AI quality control |
| `YOUR_TELEGRAM_ADMIN_CHAT_ID` | Admin chat for operational alerts |

The Supabase source table, channel name, schedule triggers, model choices, and field mappings are workflow-specific and should also be reviewed after import.

See `config/content-generation.example.json` for a deployment checklist.

## Credentials to configure in n8n

The public JSON intentionally does not contain production credentials. Configure your own credentials for:

- Supabase;
- Airtable;
- OpenAI / the selected LLM provider;
- Cloudinary;
- Telegram;
- any image-generation endpoint used by the imported workflow.

## Airtable data written by the workflow

Depending on the generation path, the workflow writes fields such as:

- post title/name;
- generated post text;
- one or more media attachments;
- content type;
- channel;
- model metadata;
- image and LLM prompts;
- system prompts;
- approximate post-generation cost.

The exact Airtable schema can be adapted to another content-plan structure.

## Quality-control handoff

The generation workflow does not publish directly. When content preparation succeeds, it calls a separate n8n workflow through an `Execute Workflow` node.

In the public export that target is represented by:

```text
YOUR_QA_WORKFLOW_ID
```

The quality-control workflow is intentionally maintained as a separate stage so generation, review, and publishing can fail, retry, and evolve independently.

## Scheduling

The exported workflow contains n8n Schedule Trigger nodes. Treat their current cron expressions as example/project-specific scheduling rather than universal defaults.

After importing the workflow:

1. choose the intended timezone;
2. review each Schedule Trigger;
3. set the desired generation cadence;
4. test both generation paths manually before enabling the workflow.

## Error handling

The workflow includes retry/error branches and Telegram notifications for selected failures, including generation/provider errors and empty-source conditions.

For a new deployment, verify that:

- the Telegram admin destination is configured;
- provider billing/quotas are valid;
- failed executions are retained in n8n;
- retry behavior is appropriate for the selected APIs.

## Import checklist

1. Import `workflows/01_content_generation.json` into n8n.
2. Bind your own credentials to every external-service node.
3. Replace all `YOUR_...` placeholders.
4. Review Supabase table and field names.
5. Review Airtable field mappings.
6. Review channel-specific prompts and channel names.
7. Set the desired schedules and timezone.
8. Configure the quality-control workflow ID.
9. Execute each branch manually with test data.
10. Enable schedules only after the full generation-to-QA handoff succeeds.

## Security

Do not commit API keys, credential IDs, private chat IDs, production webhooks, or private customer/user data.

The repository version is intended to demonstrate architecture and workflow design, not to serve as a production credential bundle.
