# Publishing Workflow

This document describes the sanitized public export in **workflows/03_publishing.json**.

## Purpose

The publishing workflow is the final stage of the pipeline. It selects approved content from Airtable, routes the record by media type, publishes it to Telegram and MAX, and writes publication timestamps back to Airtable so the same post is not sent repeatedly.

## High-level flow

~~~text
Airtable
   |
   v
Select approved post
   |
   v
Route by media type
   |
   +----------------+--------------------+
   |                |                    |
   v                v                    v
 photo            video              mix_photo
   |                |                    |
   +------ Telegram publishing ----------+
   |
   +------ MAX upload + publish ----------+
   |
   v
Update Airtable publication state
~~~

## Airtable selection

The workflow searches the content-plan table for records that have already passed quality control.

The current logic uses fields such as:

- **permission = yes** — moderation decision;
- **tg** — Telegram publication timestamp;
- **max** — MAX publication timestamp;
- **date_of_posting** — optional planned publication date;
- **Chanel** — project/channel selector;
- **type_file** — media type.

The sanitized export replaces Airtable resource identifiers with:

- **YOUR_AIRTABLE_BASE_ID**
- **YOUR_AIRTABLE_TABLE_ID**

The workflow first checks for an approved post scheduled for the current day. If no scheduled post is found, it can fall back to an approved unscheduled record.

## Supported content types

A Switch node routes the post by **type_file**.

The public workflow contains three publishing branches:

- **photo** — single image + caption;
- **video** — video + caption;
- **mix_photo** — media group / multi-image post.

## Telegram publishing

Telegram publishing is handled with native n8n Telegram nodes.

### Single image

The photo branch sends one binary image with the generated post text as an HTML caption.

### Video

The video branch sends the binary video with an HTML caption.

### Multi-image post

The mix_photo branch sends a Telegram media group with up to six images. The main caption is attached to the first media item.

After successful Telegram delivery, the workflow writes the current timestamp into the Airtable **tg** field.

This timestamp acts as publication state and prevents the record from being selected again for Telegram.

## MAX publishing

MAX publishing uses HTTP Request nodes against the MAX Platform API.

The workflow follows a multi-step upload process:

1. request an upload URL from the MAX API;
2. upload the binary media to the returned URL;
3. create the message with the uploaded media attached;
4. write the publication timestamp to Airtable.

The sanitized workflow uses **YOUR_MAX_CHAT_ID** instead of the production chat identifier.

### Single image

The workflow requests an image-upload URL, uploads the binary image, and then sends a MAX message containing the image attachment and formatted post text.

### Video

The video branch follows the same pattern using a video-upload endpoint before creating the final message.

### Multi-image post

For a multi-image post, the workflow requests and processes multiple image uploads, combines the resulting photo payloads, and creates one MAX message with the generated set of attachments.

Wait and Merge nodes are used to coordinate the individual upload steps before the final message request.

After successful MAX delivery, the workflow writes the current timestamp into the Airtable **max** field.

## Publication state

The workflow uses Airtable as the source of truth for delivery state.

~~~text
permission = yes
      |
      v
ready for publishing
      |
      +--> Telegram success --> tg = timestamp
      |
      +--> MAX success ------> max = timestamp
~~~

This makes publishing idempotent at the workflow-selection level: records with an existing publication timestamp can be excluded from subsequent runs.

## Scheduling

The exported workflow contains an n8n Schedule Trigger with project-specific cron expressions.

The schedule in a public export should be treated as an example rather than a universal deployment setting.

Before enabling the workflow:

1. select the intended n8n timezone;
2. review every cron expression;
3. confirm the content-plan publication rules;
4. test the fallback logic for unscheduled posts;
5. test each media type manually.

## Required credentials

After importing the workflow, bind your own n8n credentials for:

- Airtable;
- Telegram Bot API;
- MAX API HTTP header authentication.

The public JSON intentionally does not contain production credential objects.

## Project-specific values to review

In addition to the public placeholders, the workflow contains project-specific presentation values such as channel names, subscription links, captions, and publication filters.

For reuse in another project, review:

- Telegram chat/channel destination;
- MAX channel link;
- Airtable Chanel value;
- caption templates;
- text-length limits;
- posting schedule.

These values are not secrets, but they are specific to the original deployment.

## Setup checklist

1. Import **workflows/03_publishing.json** into n8n.
2. Bind Airtable credentials.
3. Bind Telegram credentials.
4. Bind MAX HTTP authentication credentials.
5. Replace **YOUR_AIRTABLE_BASE_ID**.
6. Replace **YOUR_AIRTABLE_TABLE_ID**.
7. Replace **YOUR_MAX_CHAT_ID**.
8. Review Telegram and MAX destinations.
9. Review Airtable publication-state fields.
10. Review schedule and timezone.
11. Test photo, video, and mix_photo separately.
12. Confirm that successful Telegram delivery writes **tg**.
13. Confirm that successful MAX delivery writes **max**.
14. Only then enable the Schedule Trigger.

## Security

Do not commit:

- API keys or bearer tokens;
- Telegram bot tokens;
- n8n credential objects;
- private chat identifiers;
- production webhook identifiers;
- private user or customer data.

The public export is intended to demonstrate the publishing architecture while keeping deployment credentials outside the repository.
