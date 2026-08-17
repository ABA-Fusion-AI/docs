---
node_id: "pixabay-image-search"
title: "Pixabay Image Search"
description: "Search for free stock images from Pixabay."
category: "Media / Stock Images"
version: "1.0.0"
language: "en"
last_updated: "2026-08-17"
author: "Fusion Team"
tags:

- pixabay
- images
- stock-photos
- media
- search
- image-search
- api

related_nodes:
- function
- if
- http-request

---

# Pixabay Image Search

> **Category:** media-nodes | **Type:** Action Node

Search for **free stock images** on **Pixabay** by keyword query.

The **Pixabay Image Search** node calls the Pixabay public API and returns a normalized list of matching photos, including multiple resolution URLs, tags, uploader info, and engagement stats.

### Supported Features

- Keyword-based image search
- Configurable results per page (up to Pixabay's 200 max)
- Safe search always enabled
- Restricted to horizontal photo orientation
- Accepts the search query from either node configuration or upstream workflow data
- Normalized result objects with multiple image resolution URLs (preview, webformat, large)
- Convenience `url` field pre-selecting the best available image URL

### Use Cases

- Find a stock photo to illustrate generated content (blog post, slide, report)
- Provide reference/source images for downstream image generation nodes
- Build a media search step in a content-creation workflow
- Populate a gallery or carousel with royalty-free images
- Feed image URLs into a `Function` or vision/LLM node for further processing

---

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `query` | `string` | ✅ Yes (unless provided via input data) | — | Search query text. |
| `perPage` | `number` | ❌ No | `5` | Number of results to return. Capped at 200 by the node (Pixabay's maximum). |
| `apiKey` | `string` | ✅ Yes | `""` | Pixabay API key. Required — the node throws if empty. |

---

## Input Data Fallback

If `query` is left empty in the configuration, the node falls back to the **incoming workflow data**: a string is used directly, and any other data type is coerced with `String(data)`.

---

## API Request Details

The node calls:

```text
GET https://pixabay.com/api/?key=<apiKey>&q=<query>&image_type=photo&orientation=horizontal&per_page=<min(perPage,200)>&safesearch=true
```

Fixed, non-configurable request parameters:

| Parameter | Fixed Value |
| --------- | ----------- |
| `image_type` | `photo` |
| `orientation` | `horizontal` |
| `safesearch` | `true` |

`per_page` is always capped at **200**, regardless of the configured `perPage` value.

---

## Inputs & Outputs

### Inputs

Optional workflow input data — used as a fallback for `query` when the config field is empty.

### Outputs

| Output | Type | Description |
| ------ | ---- | ----------- |
| `success` | `boolean` | Always `true` on a successful call. |
| `query` | `string` | The search query that was used. |
| `results` | `array` | List of normalized image objects (see below). |
| `total_results` | `number` | Number of results returned in this response (`results.length`). |
| `total_hits` | `number` | Total number of hits Pixabay has for this query (from `totalHits`), which may exceed `total_results`. |
| `note` | `string` | Always `"Use the 'url' field of each result for image generation"`. |

### Result Object Fields

| Field | Type | Description |
| ----- | ---- | ----------- |
| `id` | `number` | Pixabay image ID. |
| `url` | `string` | Best available image URL — falls back through `largeImageURL` → `webformatURL` → `previewURL` → `""`. |
| `pageURL` | `string` | Pixabay page URL for the image. |
| `tags` | `string` | Comma-separated tags describing the image. |
| `user` | `string` | Uploader's username. |
| `user_id` | `number` | Uploader's Pixabay user ID. |
| `userImageURL` | `string` | Uploader's avatar URL, or `""` if unavailable. |
| `previewURL` | `string` | Small preview thumbnail URL. |
| `previewWidth` / `previewHeight` | `number` | Preview thumbnail dimensions. |
| `webformatURL` | `string` | Medium-resolution image URL. |
| `webformatWidth` / `webformatHeight` | `number` | Medium-resolution dimensions. |
| `largeImageURL` | `string` | Large-resolution image URL. |
| `imageWidth` / `imageHeight` | `number` | Original image dimensions. |
| `downloads` | `number` | Download count on Pixabay. |
| `likes` | `number` | Like count on Pixabay. |
| `views` | `number` | View count on Pixabay. |

---

## Output Example

```json
{
  "success": true,
  "query": "mountain sunrise",
  "results": [
    {
      "id": 1234567,
      "url": "https://pixabay.com/get/large_mountain_sunrise.jpg",
      "pageURL": "https://pixabay.com/photos/mountain-sunrise-1234567/",
      "tags": "mountain, sunrise, landscape",
      "user": "naturephotos",
      "user_id": 987654,
      "userImageURL": "https://cdn.pixabay.com/user/avatar.jpg",
      "previewURL": "https://cdn.pixabay.com/photo/preview_150.jpg",
      "previewWidth": 150,
      "previewHeight": 100,
      "webformatURL": "https://pixabay.com/get/webformat_640.jpg",
      "webformatWidth": 640,
      "webformatHeight": 426,
      "largeImageURL": "https://pixabay.com/get/large_mountain_sunrise.jpg",
      "imageWidth": 4000,
      "imageHeight": 2667,
      "downloads": 12034,
      "likes": 342,
      "views": 89211
    }
  ],
  "total_results": 1,
  "total_hits": 3482,
  "note": "Use the 'url' field of each result for image generation"
}
```

---

## Configuration Examples

### Default Search

```json
{
  "query": "mountain sunrise",
  "apiKey": "your-pixabay-api-key"
}
```

### More Results Per Page

```json
{
  "query": "mountain sunrise",
  "perPage": 20,
  "apiKey": "your-pixabay-api-key"
}
```

### Maximum Results

```json
{
  "query": "mountain sunrise",
  "perPage": 200,
  "apiKey": "your-pixabay-api-key"
}
```

---

## Workflow Integration

### Sample Workflow: Search → Function

```json
{
  "nodes": [
    {
      "id": "pixabay-search",
      "type": "pixabay-image-search",
      "config": {
        "query": "mountain sunrise",
        "perPage": 10,
        "apiKey": "your-pixabay-api-key"
      }
    },
    {
      "id": "pick-best-image",
      "type": "function"
    }
  ]
}
```

### Sample Workflow: Content Generation → Pixabay → Slide/Report

```json
{
  "nodes": [
    {
      "id": "generate-topic",
      "type": "llm"
    },
    {
      "id": "pixabay-search",
      "type": "pixabay-image-search",
      "config": {
        "apiKey": "your-pixabay-api-key"
      }
    },
    {
      "id": "build-slide",
      "type": "function"
    }
  ]
}
```

Here, `pixabay-search` relies on the [input data fallback](#input-data-fallback) to receive its `query` from the upstream `generate-topic` node's output.

### Common Patterns

- LLM (generate keywords) → Pixabay Image Search → Function (select image) — automated illustration
- Pixabay Image Search → If (check `total_results`) → Notification — alert on no results
- Pixabay Image Search → Function → Database — build a reusable image library

---

## Error Handling

### Missing API Key

```text
PIXABAY_API_KEY is required
```

Raised when `apiKey` is empty.

### Missing Query

```text
Search query is required
```

Raised when both `query` and the input data are empty.

### Pixabay API Error

```text
Pixabay API error: <status>
```

Raised when Pixabay returns a non-OK HTTP status.

### Wrapped Failure

```text
Pixabay image search failed: <underlying error message>
```

All errors from `searchPixabay` (including the two above) are re-thrown wrapped in this message from `handleTick`.

---

## Troubleshooting

### "Pixabay image search failed: PIXABAY_API_KEY is required"

**Cause**

`apiKey` was left empty in the node configuration.

**Solution**

Set a valid Pixabay API key in `apiKey`.

---

### "Pixabay image search failed: Search query is required"

**Cause**

`query` is empty and no usable input data was provided.

**Solution**

Set `query` explicitly, or ensure the upstream node passes a non-empty value as input data.

---

### "Pixabay image search failed: Pixabay API error: 400"

**Cause**

The request was malformed — commonly an invalid or expired API key, or an unsupported query string.

**Solution**

Verify the API key is active and the query does not contain unsupported characters.

---

### "Pixabay image search failed: Pixabay API error: 429"

**Cause**

Pixabay's rate limit was exceeded for the given API key.

**Solution**

Reduce request frequency or wait before retrying; Pixabay enforces per-key rate limits on the free tier.

---

### `results` is Empty but `total_hits` is 0

**Cause**

No images on Pixabay match the given query under the fixed filters (`image_type=photo`, `orientation=horizontal`, `safesearch=true`).

**Solution**

Try a broader or different query — note that results are restricted to horizontal photos only, which excludes vertical images, illustrations, and vectors.

---

### `url` Field is an Empty String

**Cause**

None of `largeImageURL`, `webformatURL`, or `previewURL` were present on the Pixabay hit — unusual, but possible for certain account/content restrictions.

**Solution**

Fall back to `pageURL` to view the image directly on Pixabay, or filter out results with an empty `url` downstream.

---

## Security

The node performs outbound HTTP requests to the public Pixabay API (`pixabay.com/api`).

The `apiKey` is sent as a query parameter (`key=...`) on every request, as required by the Pixabay API.

---

## Notes

The node always applies `safesearch=true`, `image_type=photo`, and `orientation=horizontal` — these are not configurable.

The node does not:

- Support vector, illustration, or video search (photos only)
- Support vertical or square orientation
- Download or store the images itself (only returns URLs)
- Cache search results
- Retry on rate-limit (429) errors

It is intended to provide a fast, straightforward stock-photo search step for downstream content and image-generation workflows.

---

## Related

- [Function](./function.md) – Select, filter, or format search results
- [If](./if.md) – Route workflows based on result count or content
- [HTTP Request](./http-request.md) – Make additional custom Pixabay API calls
---

## Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-08-17 | Initial release |