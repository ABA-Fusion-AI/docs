---
node_id: "open-ai-dalle"
title: "OpenAI GPT Image: Generate Images"
description: "Generate images from a text prompt with the OpenAI GPT Image API and get them as base64 data and data URLs."
category: "AI / Image"
subcategory: "Image Generation"
version: "1.0.0"
language: "en"
last_updated: "2026-11-17"
author: "Fusion Team"
tags:
  - openai
  - gpt-image
  - dall-e
  - image-generation
  - ai
  - text-to-image
related_nodes:
  - openai-llm
  - ai-chat-openai
  - google-gemini-image
  - kling-ai-image-generation
---

<!-- SECTION: overview -->
# OpenAI GPT Image: Generate Images

> **Category:** AI / Image &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Generate one or more images from a text description with the **OpenAI GPT Image API** (`https://api.openai.com/v1/images/generations`). You can choose the size, quality, background transparency and output format. Each generated image is returned as **base64** data and as a ready-to-use **data URL**.

### Use Cases

- **Marketing visuals:** Generate banners, social media visuals or product illustrations from a description.
- **Content pipelines:** Create a cover image for each article or newsletter written by an LLM node.
- **Transparent assets:** Produce logos, icons or stickers with a transparent background.
- **Prototyping:** Quickly produce mockups or concept art from a list of ideas.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `apiKey` | `string` | Yes | — | OpenAI API key. Supports expressions. |
| `prompt` | `string` | Yes | — | Description of the image to generate. Supports expressions. |
| `model` | `enum` | No | `gpt-image-2` | GPT Image model used to generate images. |
| `n` | `number` | No | `1` | Number of images to generate (integer from 1 to 10). |
| `size` | `enum` | No | `1024x1024` | `1024x1024` (square), `1536x1024` (landscape) or `1024x1536` (portrait). |
| `quality` | `enum` | No | `auto` | `auto`, `low`, `medium` or `high`. Higher quality is slower and costs more. |
| `background` | `enum` | No | `auto` | `auto`, `opaque` or `transparent`. |
| `outputFormat` | `enum` | No | `png` | `png`, `jpeg` or `webp`. |

> A `transparent` background requires the `png` or `webp` format; `jpeg` does not support transparency.

### Authentication & API Key

1. Log in to the **OpenAI Platform** (`https://platform.openai.com`).
2. Open **API keys** and click **Create new secret key**.
3. Copy the key and paste it into the `apiKey` field (or store it in Fusion Secrets).

> Image generation is billed per image and depends on `quality` and `size`. Some organizations must be verified on the OpenAI Platform before using GPT Image models.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` | Triggers the node. Use upstream values in `prompt` through expressions. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Model, number of images and the generated images. |
| `error` | `Error` | Emitted on validation errors, API errors or empty responses. |

### Output Fields

| Field | Type | Description |
|-------|------|-------------|
| `model` | `string` | Model used. |
| `count` | `number` | Number of images returned. |
| `images[].index` | `number` | Position of the image (starts at `0`). |
| `images[].format` | `string` | Output format (`png`, `jpeg`, `webp`). |
| `images[].mimeType` | `string` | MIME type, e.g. `image/png`. |
| `images[].base64` | `string` | Image encoded in base64. |
| `images[].dataUrl` | `string` | `data:image/...;base64,...` URL, usable directly in HTML or email. |
| `images[].revisedPrompt` | `string \| null` | Prompt rewritten by the model, when provided. |

### Output Example

```json
{
  "model": "gpt-image-2",
  "count": 1,
  "images": [
    {
      "index": 0,
      "format": "png",
      "mimeType": "image/png",
      "base64": "iVBORw0KGgoAAAANSUhEUgAABAAAAAQACAIAAADwf7zU...",
      "dataUrl": "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAABAAAAAQACAIAAADwf7zU...",
      "revisedPrompt": null
    }
  ]
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Generate an image with GPT Image
```

### How it flows

1. **Manual Trigger:** Starts the workflow.
2. **OpenAI GPT Image Node:** Sends the prompt to the OpenAI GPT Image API.
3. **Log Node:** Displays the generated image data.

### Example Configuration

```json
{
  "apiKey": "{{ secrets.OPENAI_API_KEY }}",
  "prompt": "A minimalist flat illustration of a robot watering a plant, pastel colors",
  "model": "gpt-image-2",
  "n": 1,
  "size": "1536x1024",
  "quality": "medium",
  "background": "auto",
  "outputFormat": "png"
}
```

> To save the image as a file, pass `images[0].base64` to a file node; to show it in an email or HTML page, use `images[0].dataUrl`.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: security -->
## Security

> Store your OpenAI `apiKey` in Fusion's **Secrets** vault. Do not commit hardcoded keys into version control.

- Set a monthly usage limit on your OpenAI account to control image generation costs.
- Generated images are returned in the workflow data: avoid logging large base64 payloads in production.

<!-- /SECTION: security -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `OpenAI API key is required.` / `Image prompt is required.`
- **Cause:** `apiKey` or `prompt` is empty.
- **Solution:** Fill in both fields.

#### `The number of images must be an integer between 1 and 10.`
- **Cause:** `n` is lower than 1, higher than 10, or not an integer.
- **Solution:** Set `n` to an integer between 1 and 10.

#### `OpenAI API error (Status: 401): Invalid API key...`
- **Cause:** The API key is wrong, revoked or expired.
- **Solution:** Create a new key on the OpenAI Platform.

#### `OpenAI API error (Status: 403): Access denied...`
- **Cause:** The organization has no access to the model (for example, organization not verified).
- **Solution:** Check the organization's verification and model access in the OpenAI Platform settings.

#### `OpenAI API error (Status: 429): Rate limit or quota exceeded...`
- **Cause:** Too many requests, or no credit left.
- **Solution:** Wait before retrying and check usage and billing.

#### `OpenAI API error (Status: 400): Invalid request parameters...`
- **Cause:** Invalid parameter combination (e.g. `transparent` background with `jpeg`), or the prompt was rejected by the safety system.
- **Solution:** Use `png`/`webp` for transparency, and rephrase the prompt if it was blocked.

#### `OpenAI API returned a successful response but no images were generated.`
- **Cause:** The API answered without image data.
- **Solution:** Retry the request.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-11-17 | Initial documentation: parameter reference, output schema, and workflow integration |

<!-- /SECTION: changelog -->
