# AI Quality Control Workflow

This document describes the sanitized public export in `workflows/02_ai_quality_control.json`.

## Purpose

The workflow acts as a pre-publication quality gate for prepared content. It reads unreviewed records from Airtable, applies deterministic validation first, then uses AI moderation for text and, where needed, images. The result is written back to Airtable as an approval decision and a review comment.

## High-level flow

```text
Airtable
   |
   v
Select unreviewed post
   |
   v
Route by content type
   |
   +-------------------------+
   |                         |
   v                         v
photo                    mix_photo
   |                         |
Check image exists       Check all 6 images exist
   |                         |
Check text length        Check text length
   |                         |
AI text review           AI image review
   |                         |
Parse JSON               If image check passes
   |                         |
Write decision           AI text review
                             |
                             v
                         Parse JSON
                             |
                             v
                         Write decision
```

## Workflow entry points

The exported workflow can be started in two ways:

- by an n8n Schedule Trigger;
- by another n8n workflow through the Execute Workflow trigger.

This allows the quality-control stage to run independently or to be called immediately after content generation.

## Airtable selection

The workflow searches the Airtable content plan for one record where the moderation decision has not yet been written.

The public export keeps the Airtable resources as placeholders:

- `YOUR_AIRTABLE_BASE_ID`
- `YOUR_AIRTABLE_TABLE_ID`

The current project-specific filter also targets the configured channel and should be adapted for another deployment.

## Content routing

A Switch node routes records by `type_file`.

The current workflow contains dedicated logic for:

- `photo` — one image plus post text;
- `mix_photo` — a multi-image post using six image fields.

## Deterministic checks

Before calling an LLM, the workflow performs hard validation in n8n.

### Single-image posts

The workflow checks that:

1. the main image URL exists;
2. the post text is not empty after stripping HTML;
3. the text length is within the configured 1024-character limit.

If a required image is missing, the workflow rejects the record and writes a failure reason to Airtable.

If the text does not meet the limit, the record is rejected before AI moderation.

### Multi-image posts

For the multi-image branch, the workflow checks that all six expected image URLs exist and that the post text satisfies the same length validation.

This keeps simple, deterministic rules outside the LLM.

## AI text moderation

The text-moderation agent checks the post for publication readiness.

The current prompt validates:

- Russian spelling and grammar;
- readability and logical consistency;
- broken phrases, repetitions, strange symbols, and meaningless fragments;
- HTML correctness;
- use of the allowed `<b>` and `<i>` tags;
- absence of unsupported list tags and escaped HTML markup;
- suitability for the target Telegram content format.

The agent separates findings into critical and non-critical issues.

The expected structured decision is conceptually:

```json
{
  "permission": "yes",
  "komment": "без замечаний"
}
```

A critical issue results in `permission = no`. Non-critical stylistic remarks may still allow publication.

## AI image moderation

The multi-image branch also performs image analysis before the final text decision.

The image check looks for issues such as:

- obvious AI artifacts;
- distorted food, hands, dishes, or ingredients;
- visual clutter or unexpected objects;
- watermarks, logos, or advertising text;
- visible text errors;
- inconsistency between the images and the recipe;
- lack of continuity across the step-by-step sequence.

The image-review output is parsed into a structured approval decision and comment.

If the image stage does not approve the content, the workflow can stop before the text-review result is accepted.

## Fail-closed parsing

AI responses are parsed by Code nodes rather than being trusted as free-form text.

If the text-review agent returns invalid JSON, the parser defaults to:

```text
permission = no
```

and records that manual review is required.

The image parser follows the same fail-closed principle when a JSON response is missing or cannot be parsed.

This prevents malformed model output from silently approving content.

## Airtable output

The quality-control stage writes the result back to the same Airtable record.

The important fields are:

- `permission` — final moderation decision;
- `grade_koment` — AI or deterministic validation comment;
- `grade` — used in selected rejection branches.

For multi-image content, the final comment can include both text-review and image-review feedback.

## Models

The public export contains OpenAI model nodes used for text and image analysis.

Model selection is part of the imported n8n workflow and can be changed for another deployment without changing the overall architecture.

## Setup checklist

1. Import `workflows/02_ai_quality_control.json` into n8n.
2. Bind your own Airtable credentials.
3. Bind your own OpenAI/LLM credentials.
4. Replace `YOUR_AIRTABLE_BASE_ID`.
5. Replace `YOUR_AIRTABLE_TABLE_ID`.
6. Review the Airtable field names and channel filter.
7. Review the deterministic text-length limit.
8. Review moderation prompts for the target content type.
9. Test rejection paths with missing media and invalid text.
10. Test valid and invalid AI JSON responses.
11. Test both `photo` and `mix_photo` branches.
12. Only then enable the schedule or connect the workflow to the generation stage.

## Security

The public workflow does not include production credentials.

Do not commit API keys, n8n credential objects, private user data, private webhook identifiers, or production-only resource identifiers.

The sanitized export is intended to demonstrate the moderation architecture and state flow while keeping deployment-specific secrets outside the repository.
