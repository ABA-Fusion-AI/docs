---
node_id: "stability-ai"
title: "Stability AI: Generate Images"
description: "Generate photorealistic images and perform AI image-to-image synthesis using the Stability AI Ultra API."
category: "ai"
subcategory: "image-generation"
version: "1.0.0"
language: "en"
last_updated: "2026-11-24"
author: "Fusion Team"
tags:
  - stability-ai
  - stable-diffusion
  - image-generation
  - text-to-image
  - image-to-image
  - ai
  - generative-ai
related_nodes:
  - function
  - manual-trigger
---

<!-- SECTION: overview -->
# Stability AI: Generate Images

> **Category:** Generative AI&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

The **Stability AI: Generate Images** node connects workflows directly to the flagship **Stability AI Ultra API** (`/v2beta/stable-image/generate/ultra`). It provides enterprise-grade text-to-image synthesis and image-to-image transformation with photorealistic lighting, complex typography rendering, and prompt adherence.

The node supports configurable aspect ratios, deterministic generation via random seeds, multiple output image formats (PNG, JPEG, WebP), and automatic handling of remote input images (HTTP/HTTPS URLs, data URIs, and raw base64 buffers). All generated media includes standard C2PA Content Credentials metadata.

### Use Cases

- **Marketing & Creative Asset Creation:** Generate high-resolution visuals, banners, and concept art from structured prompt pipelines.
- **Product Re-Styling & Mockups:** Transform existing product photos into different styles, artistic media, or environments using `image-to-image`.
- **E-Commerce Visual Variation:** Generate branded backgrounds and seasonal scene variations from reference images with controlled `strength`.
- **Automated Social Content:** Dynamically create tailored illustrations based on incoming article summaries, headlines, or RSS triggers.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Basic Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `apiKey` | `string` | ✅ Yes | — | Stability AI API key (format: `sk-...`). |
| `prompt` | `string` | ✅ Yes | — | Detailed text description of the image to generate. Supports upstream expressions. |
| `mode` | `enum` | ❌ No | `text-to-image` | Synthesis mode:<br>‑ `text-to-image`: Synthesize a new image purely from text.<br>‑ `image-to-image`: Modify an existing reference image based on text and strength. |
| `aspectRatio` | `enum` | ❌ No | — | Target image proportions for `text-to-image` mode:<br>‑ `1:1` (Square)<br>‑ `16:9`, `21:9` (Widescreen/Cinematic landscape)<br>‑ `9:16`, `9:21` (Vertical/Stories/Reels portrait)<br>‑ `3:2`, `5:4` (Standard photo landscape)<br>‑ `2:3`, `4:5` (Standard photo portrait) |
| `outputFormat` | `enum` | ❌ No | `png` | File format of the synthesized image:<br>‑ `png`: Lossless quality with transparency support.<br>‑ `jpeg`: Compressed photo format (magic bytes `/9j/`).<br>‑ `webp`: Highly optimized web format. |
| `seed` | `number` | ❌ No | — | Integer between `0` and `4294967294`. Using the same seed with identical parameters produces consistent, deterministic visual outputs. |
| `image` | `string` | ⚠️ Conditional | — | **Required when `mode` is `image-to-image`.** The reference image to transform. Supports remote URLs (`https://...`), Data URIs (`data:image/png;base64,...`), or raw base64 strings. |
| `strength` | `number` | ❌ No | `0.5` | Transformation intensity for `image-to-image` (range `0.0` to `1.0`). `0.0` leaves the original unchanged; `1.0` completely reimagines the scene. |

---

### Image Input Methods (`image-to-image`)

The node automatically detects and processes three forms of input in the `image` field:

1. **Remote HTTP/HTTPS URLs:** The node downloads the image buffer in memory, validates that the HTTP `Content-Type` header matches an image MIME type (`image/png`, `image/jpeg`, `image/webp`), and uploads it as a multipart Blob.
2. **Data URIs:** Strings beginning with `data:image/...;base64,...` are regex-validated, decoded from base64, and packaged with their declared MIME type.
3. **Raw Base64:** Unprefixed base64 image strings are converted into PNG image blobs automatically.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Incoming data payload. Can supply dynamic prompts, aspect ratios, or source image URLs via expression bindings. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted when image synthesis completes successfully. |
| `error` | `object` | Emitted when parameter validation fails, network requests error, or API quotas are exceeded. |

---

### Output Schema (`success`)

Upon successful generation, the node returns an object containing the base64-encoded image:

```json
{
  "image": "iVBORw0KGgoAAAANSUhEUgAABUAAAAMACAIAAABq7Fo6...",
  "finish_reason": "SUCCESS",
  "seed": 42
}
```

- `image`: Base64-encoded binary string. Can be used in downstream nodes (e.g. uploaded to S3, emailed as attachment, or displayed in a dashboard).
- `finish_reason`: Set to `SUCCESS` when the generation reaches completion without content moderation filters triggering.
- `seed`: The actual seed used for this generation.

---

### Output Schema (`error`)

If an error occurs (such as validation failures or insufficient credits), the node returns a structured error:

```json
{
  "name": "Error",
  "message": "Stability AI API error (Status: 402): You lack sufficient credits to make this request. Please purchase more credits at https://platform.stability.ai/dashboard/billing and try again."
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
### Example Workflow

The following workflow demonstrates dynamically assembling a creative prompt and aspect ratio inside a **Function** node, and generating an ultra-high-definition image with **Stability AI: Generate Images**.

```fusion-workflow
src: example.workflow.json
title: Generate Images with Stability AI
```

### Step-by-Step Walkthrough

1. **Manual Trigger:** Triggers the workflow execution on demand.
2. **Function Node (`Prepare Prompt`):** Computes a dynamic prompt and preferred aspect ratio:
   ```javascript
   return {
     aiPrompt: "A futuristic robotic Falcon with golden wings, octane render, 8k",
     ratio: "16:9"
   };
   ```
3. **Stability AI Node:**
   - Evaluates `prompt` dynamically from `outputs.Function.success.aiPrompt`.
   - Evaluates `aspectRatio` dynamically from `outputs.Function.success.ratio`.
   - Dispatches a multipart request to Stability AI Ultra API with fixed seed `42`.
   - Receives the synthesized base64 image ready for downstream consumers.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Stability AI API error (Status: 401): authorization: invalid or missing header value`

**Cause:** The provided `apiKey` is empty, malformed, or has been revoked.

**Solution:** Generate a valid API key from your [Stability AI Account](https://platform.stability.ai/account/keys) and ensure the string begins with `sk-`.

#### `Stability AI API error (Status: 402): You lack sufficient credits to make this request`

**Cause:** Your Stability AI account has exhausted its available credit balance. The Ultra model consumes approximately 8 credits per generation.

**Solution:** Top up credits in the Stability AI Billing Dashboard.

#### `Stability AI Error: 'image' is required when mode is 'image-to-image'`

**Cause:** `mode` was set to `image-to-image`, but the `image` field was left blank.

**Solution:** Supply an image URL or base64 string in the `image` parameter, or change `mode` to `text-to-image`.

#### `Input image URL did not return an image content type`

**Cause:** The URL provided in `image` returned a non-image content type (such as HTML `text/html` from a webpage).

**Solution:** Ensure the URL points directly to an image asset ending in `.png`, `.jpg`, `.jpeg`, or `.webp`.

#### `Unable to download input image: 404 Not Found`

**Cause:** The remote image URL is broken, inaccessible, or has been deleted.

**Solution:** Verify that the image URL is publicly accessible without authentication.

#### `Input image data URL must contain a base64 PNG, JPEG, or WebP image`

**Cause:** A Data URI was provided with an unsupported MIME type (e.g. `data:text/plain;...`).

**Solution:** Ensure data URIs follow the standard format: `data:image/png;base64,<data>`, `data:image/jpeg;base64,<data>`, or `data:image/webp;base64,<data>`.

### Error Codes

| Error Message | HTTP Status | Cause | Resolution |
|---------------|:-----------:|-------|------------|
| `authorization: invalid or missing header value` | `401` | Invalid API key | Verify and replace `apiKey` |
| `You lack sufficient credits` | `402` | Out of credits | Top up credits on Stability AI platform |
| `'image' is required when mode is 'image-to-image'` | N/A | Missing reference image | Provide image URL or switch to `text-to-image` |
| `Unable to download input image: 404` | `404` | Broken image URL | Check source image link |
| `Input image URL did not return an image content type` | N/A | Non-image URL supplied | Provide direct link to an image file |
| `Input image data URL must contain a base64...` | N/A | Malformed Data URI | Format data URI as valid base64 image |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Function](/docs/function) – Prepare dynamic prompts, styles, and parameters
- [Manual Trigger](/docs/manual-trigger) – Manually trigger generative workflows

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-11-24 | Initial release of Stability AI Generate Images action node documentation |

<!-- /SECTION: changelog -->
