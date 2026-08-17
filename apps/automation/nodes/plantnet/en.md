---
node_id: "plant-net"
title: "Pl@ntNet ID"
description: "Identify plant species from photos."
category: "Image Recognition"
version: "1.0.0"
language: "en"
last_updated: "2026-08-17"
author: "Fusion Team"
tags:

- plantnet
- plant-identification
- image-recognition
- botany
- species-identification
- computer-vision
- api

related_nodes:
- function
- if
- http-request

---

# Pl@ntNet ID

> **Category:** image-recognition-nodes | **Type:** Action Node

Identify **plant species** from a photo using the **Pl@ntNet** visual identification API.

The **Pl@ntNet ID** node downloads an image from a public URL, uploads it to Pl@ntNet as multipart form data, and returns the best-matching species along with the full ranked result set.

### Supported Features

- Plant species identification from a public image URL
- Organ selection (`leaf`, `flower`, `fruit`, `bark`, or `auto`) to improve match accuracy
- Optional inclusion of reference images for the matched species
- Multipart image upload (image bytes, not just the URL, are sent to Pl@ntNet)
- Simplified "best match" summary alongside the full raw API response

### Use Cases

- Identify a plant species from a photo submitted through a form or chat
- Build a citizen-science or gardening assistant workflow
- Auto-tag uploaded plant images with scientific and common names
- Validate or enrich plant/crop data in an agricultural workflow
- Feed identification results into a notification or database node

---

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `apiKey` | `string` | ✅ Yes | — | Pl@ntNet API key. Must be at least 1 character. |
| `imageUrl` | `string` | ✅ Yes | — | Public URL of the plant image to identify. Must be at least 1 character. |
| `organ` | `enum` | ❌ No | `"auto"` | Which part of the plant is shown: `leaf`, `flower`, `fruit`, `bark`, or `auto`. |
| `includeRelatedImages` | `boolean` | ❌ No | `true` | Whether to request reference images of the matched species. |

---

## How It Works

1. **Download** — the node fetches the image bytes from `imageUrl`. The image is downloaded server-side rather than passing the URL directly to Pl@ntNet, since Pl@ntNet is more reliable with actual file uploads.
2. **Build multipart form** — the downloaded image is attached as `images` (filename `plant.jpg`), and the selected `organ` is attached as `organs`.
3. **Submit** — a `POST` request is sent to Pl@ntNet's `/v2/identify/all` endpoint with the API key and `include-related-images` flag as query parameters.
4. **Simplify** — the top result (`data.results[0]`, if any) is extracted into a `bestMatch` summary, alongside the complete `fullResults` array.

---

## Inputs & Outputs

### Inputs

The node does not read workflow input data directly — `imageUrl` must be set in the node configuration.

### Outputs

| Output | Type | Description |
| ------ | ---- | ----------- |
| `bestMatch` | `object \| string` | Simplified top match (see below), or the string `"No match found"` if Pl@ntNet returned no results. |
| `bestMatch.scientificName` | `string` | Scientific name without author, for the top match. |
| `bestMatch.commonNames` | `string[]` | Common names for the top match species. |
| `bestMatch.score` | `number` | Confidence score for the top match (0–1). |
| `fullResults` | `array` | Full raw `results` array from the Pl@ntNet API response, containing all candidate matches with scores, species details, and (if requested) reference images. |

---

## Output Example

```json
{
  "bestMatch": {
    "scientificName": "Rosa canina L.",
    "commonNames": ["Dog rose", "Briar rose"],
    "score": 0.87234
  },
  "fullResults": [
    {
      "score": 0.87234,
      "species": {
        "scientificNameWithoutAuthor": "Rosa canina",
        "scientificNameAuthorship": "L.",
        "commonNames": ["Dog rose", "Briar rose"],
        "family": { "scientificNameWithoutAuthor": "Rosaceae" },
        "genus": { "scientificNameWithoutAuthor": "Rosa" }
      },
      "images": [
        {
          "url": { "o": "https://bs.plantnet.org/image/o/..." },
          "organ": "flower"
        }
      ]
    }
  ]
}
```

### No Match Found

```json
{
  "bestMatch": "No match found",
  "fullResults": []
}
```

---

## Configuration Examples

### Default (Auto-Detect Organ)

```json
{
  "apiKey": "your-plantnet-api-key",
  "imageUrl": "https://example.com/photos/plant.jpg"
}
```

### Leaf-Based Identification

```json
{
  "apiKey": "your-plantnet-api-key",
  "imageUrl": "https://example.com/photos/leaf-closeup.jpg",
  "organ": "leaf"
}
```

### Flower Identification Without Reference Images

```json
{
  "apiKey": "your-plantnet-api-key",
  "imageUrl": "https://example.com/photos/flower.jpg",
  "organ": "flower",
  "includeRelatedImages": false
}
```

---

## Workflow Integration

### Sample Workflow: Identify → Function

```json
{
  "nodes": [
    {
      "id": "plantnet-id",
      "type": "plantnet-id",
      "config": {
        "apiKey": "your-plantnet-api-key",
        "imageUrl": "https://example.com/photos/plant.jpg",
        "organ": "leaf"
      }
    },
    {
      "id": "format-result",
      "type": "function"
    }
  ]
}
```

### Sample Workflow: Identify → If → Notification

```json
{
  "nodes": [
    {
      "id": "plantnet-id",
      "type": "plantnet-id",
      "config": {
        "apiKey": "your-plantnet-api-key",
        "imageUrl": "https://example.com/photos/plant.jpg"
      }
    },
    {
      "id": "check-confidence",
      "type": "if"
    },
    {
      "id": "notify-result",
      "type": "notification"
    }
  ]
}
```

### Sample Workflow: Identify → Database

```json
{
  "nodes": [
    {
      "id": "plantnet-id",
      "type": "plantnet-id",
      "config": {
        "apiKey": "your-plantnet-api-key",
        "imageUrl": "https://example.com/photos/plant.jpg"
      }
    },
    {
      "id": "store-identification",
      "type": "database"
    }
  ]
}
```

### Common Patterns

- Chat/Form upload → Pl@ntNet ID → Function → Notification — user-facing identification bot
- Pl@ntNet ID → If (score threshold) → Database — only store confident matches
- Pl@ntNet ID → Function → Map/visualization pipeline — plot identified species by location

---

## Error Handling

### Image Download Failure

```text
Failed to download image: <imageUrl>
```

Raised when the `imageUrl` cannot be fetched (non-OK HTTP response).

### Pl@ntNet API Error

```text
Pl@ntNet Error (<status>): <message>
```

Raised when Pl@ntNet returns a non-OK HTTP status. The message is taken from the response body's `message` or `error` field.

---

## Troubleshooting

### "Failed to download image: <imageUrl>"

**Cause**

The `imageUrl` is unreachable, private, requires authentication, or does not point directly to an image file.

**Solution**

Verify the URL is publicly accessible (no login wall) and resolves directly to an image (not an HTML page).

---

### "Pl@ntNet Error (401): ..." or similar authentication error

**Cause**

The `apiKey` is invalid, expired, or missing required permissions.

**Solution**

Verify the API key in the Pl@ntNet developer account and confirm it has not been revoked or rate-limited.

---

### "Pl@ntNet Error (404): ..." or unexpected identification failure

**Cause**

The submitted image was not recognized as containing identifiable plant material, or the request format was rejected.

**Solution**

Confirm the image clearly shows a plant organ matching the selected `organ` value, and try `organ: "auto"` if uncertain.

---

### `bestMatch` is `"No match found"`

**Cause**

Pl@ntNet processed the request successfully but returned zero candidate species — typically due to poor image quality, an obscured subject, or a plant outside Pl@ntNet's reference database.

**Solution**

Try a clearer photo, a different organ selection, or a close-up of a more distinctive plant feature (e.g. flower instead of leaf).

---

### Low `score` on `bestMatch`

**Cause**

Pl@ntNet's confidence in the top match is low — the photo may be ambiguous or the species may be visually similar to others.

**Solution**

Inspect `fullResults` for other close-scoring candidates, or request a better photo before accepting the identification.

---

## Security

The node performs two outbound HTTP calls: one to download the image at `imageUrl`, and one `POST` to Pl@ntNet's API (`my-api.plantnet.org`).

The `apiKey` is sent as a query parameter (`api-key=...`) on the Pl@ntNet request, as required by the Pl@ntNet API.

Since `imageUrl` is a user/workflow-supplied URL that the node fetches server-side, only trusted or validated URLs should be passed in to avoid unintended outbound requests (SSRF-style exposure).

---

## Notes

The node returns both a simplified `bestMatch` summary and the complete `fullResults` array, so downstream nodes can access secondary candidates if needed.

The node does not:

- Accept a directly uploaded image file (only a public URL)
- Cache identification results
- Validate that `imageUrl` points to a plant photo before uploading
- Support batch identification of multiple images in a single call
- Store or persist identification history

It is intended to provide a single-image, single-call plant identification step for downstream workflow processing.

---

## Related

- [Function](./function.md) – Transform or format identification results
- [If](./if.md) – Route workflows based on match score or species
- [HTTP Request](./http-request.md) – Make additional custom Pl@ntNet API calls

---

## Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-08-17 | Initial release |