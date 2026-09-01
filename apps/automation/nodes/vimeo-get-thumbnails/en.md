---
node_id: "vimeo-get-thumbnails"
title: "Vimeo Get Thumbnails"
description: "Get Vimeo video thumbnails. Tries oEmbed API first (no auth), then falls back to Vimeo API if token provided."
category: "Media / Video"
version: "1.0.0"
language: "en"
last_updated: "2026-09-01"
author: "Fusion Team"
tags:

- vimeo
- thumbnails
- oembed
- media
- images
- api

related_nodes:
- vimeo-get-video
- vimeo-get-transcript
- function
- if

---

# Vimeo Get Thumbnails

> **Category:** media-nodes | **Type:** Action Node

Get **thumbnail image URLs** for a Vimeo video, trying the **oEmbed API first (no authentication required)**, and falling back to the **authenticated Vimeo API** if an access token is provided and oEmbed doesn't succeed.

The **Vimeo Get Thumbnails** node follows the same two-tier lookup pattern as [Vimeo Get Video](./vimeo-get-video.md): an unauthenticated oEmbed attempt first, then an authenticated API fallback — but the two paths produce **meaningfully different thumbnail data**, not just a differently-shaped version of the same information (see below).

## ⚠️ Important: oEmbed Only Provides ONE Thumbnail URL, Duplicated Three Times

When the oEmbed path succeeds, it only has access to a **single** thumbnail URL (`oembedData.thumbnail_url`). The node assigns this **same URL** to all three of `thumbnail_small`, `thumbnail_medium`, and `thumbnail_large`. This means that under `method: "oembed"`, requesting a "large" vs "small" thumbnail returns the **identical image URL** — there is no actual size variation unless the authenticated API path was used instead.

### Supported Features

- Video ID extraction from several Vimeo URL formats (same patterns as other Vimeo nodes in this series)
- Primary lookup via Vimeo's public oEmbed API — no authentication needed (single thumbnail size only, see warning)
- Automatic fallback to the authenticated Vimeo API (`/videos/{id}/pictures`) if oEmbed fails and an `accessToken` is provided (multiple genuine sizes available)
- Optional single-size selection via `thumbnailSize`, alongside the full map of all discovered thumbnails
- Reports which method (`oembed` or `api`) actually produced the result

---

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `videoUrl` | `string` | ✅ Yes | — | Vimeo video URL (or a bare numeric video ID). Must be non-empty. |
| `accessToken` | `string` | ❌ No | — | Vimeo API access token, used only as a fallback if oEmbed fails to produce any thumbnail. |
| `thumbnailSize` | `enum` | ❌ No | `"all"` | `"all"`, `"thumbnail_small"`, `"thumbnail_medium"`, or `"thumbnail_large"`. Selects a single URL to surface in `selectedThumbnail`, in addition to the full `thumbnails` map. |

---

## Video ID Extraction

Identical logic to [Vimeo Get Video](./vimeo-get-video.md#video-id-extraction): matches standard Vimeo URLs (including channel/group/album variants) or a bare numeric ID. Throws `"Invalid Vimeo URL. Could not extract video ID."` if neither pattern matches.

---

## Lookup Strategy

1. **Try oEmbed** (`https://vimeo.com/api/oembed.json?url=<videoUrl>`) — no authentication. If `oembedData.thumbnail_url` is present, it is assigned to **all three** keys (`thumbnail_small`, `thumbnail_medium`, `thumbnail_large`) in the `thumbnails` map, and `method` is set to `"oembed"`.
2. **If `thumbnails` is still empty after oEmbed** (no thumbnail URL in the response, or the request failed/threw) **and `accessToken` is set and non-blank** — try `GET https://api.vimeo.com/videos/{videoId}/pictures` with the token. This response can contain **multiple real sizes** — every `size.type` found across every picture entry is added to the `thumbnails` map (e.g. `"100x75"`, `"640"`, `"960"`, `"1280"`, or whatever type strings Vimeo's Pictures API returns for that video). `method` is set to `"api"`.
3. **If `thumbnails` is still empty after both attempts** — the node throws, noting whether the API fallback was attempted.

Both attempts wrap their logic in try/catch blocks that **silently swallow errors** — no error detail from either failed attempt is retained or surfaced.

---

## ⚠️ Thumbnail Key Naming Differs Between Methods

Because of how each method populates `thumbnails`, the **keys present in the map depend entirely on `method`**:

| `method` | Typical Keys in `thumbnails` |
| ---------- | ------------------------------ |
| `"oembed"` | Exactly `thumbnail_small`, `thumbnail_medium`, `thumbnail_large` — all pointing to the same URL. |
| `"api"` | Whatever `size.type` strings the Vimeo Pictures API returns for that video — these are **not** guaranteed to match the `thumbnail_small`/`medium`/`large` naming at all (they may be numeric width strings like `"100x75"`). |

This has a direct consequence for `thumbnailSize` selection — see below.

---

## `thumbnailSize` Selection Behavior

When `thumbnailSize` is not `"all"`:

```text
selectedThumbnail = thumbnails[thumbnailSize] ?? Object.values(thumbnails)[0]
result.selectedSize = thumbnailSize   // always echoes the requested value, even on fallback
```

If the requested key (e.g. `"thumbnail_large"`) doesn't exist in `thumbnails` — which is the **normal case** when `method: "api"`, since the API's size keys typically don't match the `thumbnail_*` naming — the node silently falls back to the **first value in the map** (in insertion order), while still reporting `selectedSize` as the originally requested value. This can be misleading: `selectedSize: "thumbnail_large"` does not guarantee the returned `selectedThumbnail` is actually a large image.

---

## Inputs & Outputs

### Inputs

The node does not require workflow input. All configuration is provided through the node configuration.

### Outputs

| Output | Type | Description |
| ------ | ---- | ----------- |
| `success` | `boolean` | Always `true` if at least one thumbnail was found (the node throws instead of returning `success: false`). |
| `videoUrl` | `string` | The input video URL. |
| `videoId` | `string` | Extracted numeric video ID. |
| `thumbnails` | `object` | Map of size-key → thumbnail URL. Keys depend on `method` (see above). |
| `method` | `string` | `"oembed"` or `"api"` — which lookup path produced the thumbnails. |
| `selectedThumbnail` | `string` | Only present if `thumbnailSize !== "all"` — the URL selected per [`thumbnailSize` Selection Behavior](#thumbnailsize-selection-behavior). |
| `selectedSize` | `string` | Only present if `thumbnailSize !== "all"` — echoes the requested `thumbnailSize`, regardless of whether an exact key match was found. |

---

## Output Example

### `method: "oembed"`, `thumbnailSize: "all"`

```json
{
  "success": true,
  "videoUrl": "https://vimeo.com/123456789",
  "videoId": "123456789",
  "thumbnails": {
    "thumbnail_medium": "https://i.vimeocdn.com/video/thumb_640.jpg",
    "thumbnail_large": "https://i.vimeocdn.com/video/thumb_640.jpg",
    "thumbnail_small": "https://i.vimeocdn.com/video/thumb_640.jpg"
  },
  "method": "oembed"
}
```

### `method: "api"`, `thumbnailSize: "all"`

```json
{
  "success": true,
  "videoUrl": "https://vimeo.com/123456789",
  "videoId": "123456789",
  "thumbnails": {
    "100x75": "https://i.vimeocdn.com/video/thumb_100x75.jpg",
    "640": "https://i.vimeocdn.com/video/thumb_640.jpg",
    "1280": "https://i.vimeocdn.com/video/thumb_1280.jpg"
  },
  "method": "api"
}
```

### `method: "api"`, `thumbnailSize: "thumbnail_large"` (key not present — fallback triggered)

```json
{
  "success": true,
  "videoUrl": "https://vimeo.com/123456789",
  "videoId": "123456789",
  "thumbnails": {
    "100x75": "https://i.vimeocdn.com/video/thumb_100x75.jpg",
    "640": "https://i.vimeocdn.com/video/thumb_640.jpg"
  },
  "method": "api",
  "selectedThumbnail": "https://i.vimeocdn.com/video/thumb_100x75.jpg",
  "selectedSize": "thumbnail_large"
}
```

---

## Configuration Examples

### All Thumbnails, No Token (oEmbed Only)

```json
{
  "videoUrl": "https://vimeo.com/123456789"
}
```

### With Fallback Token (Real Multiple Sizes)

```json
{
  "videoUrl": "https://vimeo.com/123456789",
  "accessToken": "your-vimeo-access-token"
}
```

### Request a Specific Size

```json
{
  "videoUrl": "https://vimeo.com/123456789",
  "thumbnailSize": "thumbnail_large"
}
```

---

## Workflow Integration

### Sample Workflow: Get Thumbnails → Function

```json
{
  "nodes": [
    {
      "id": "vimeo-thumbnails",
      "type": "vimeo-get-thumbnails",
      "config": {
        "videoUrl": "https://vimeo.com/123456789"
      }
    },
    {
      "id": "build-thumbnail-gallery",
      "type": "function"
    }
  ]
}
```

### Sample Workflow: Get Video + Get Thumbnails (parallel enrichment)

```json
{
  "nodes": [
    {
      "id": "vimeo-get-video",
      "type": "vimeo-get-video",
      "config": {
        "videoUrl": "https://vimeo.com/123456789"
      }
    },
    {
      "id": "vimeo-thumbnails",
      "type": "vimeo-get-thumbnails",
      "config": {
        "videoUrl": "https://vimeo.com/123456789",
        "accessToken": "your-vimeo-access-token"
      }
    },
    {
      "id": "merge-video-metadata",
      "type": "function"
    }
  ]
}
```

### Common Patterns

- Vimeo Get Thumbnails → Function → Preview card/embed rendering
- Vimeo Get Video + Vimeo Get Thumbnails → Function → combined metadata for a video listing UI
- Vimeo Get Thumbnails (with token) → Database — archive real multi-size thumbnails for later use

---

## Error Handling

### Missing Video URL

```text
Video URL is required
```

Raised when `videoUrl` is empty or whitespace-only.

### Invalid URL

```text
Failed to get Vimeo thumbnails: Invalid Vimeo URL. Could not extract video ID.
```

### Both Lookups Failed

```text
Failed to get Vimeo thumbnails: Failed to get thumbnails. Tried oEmbed API.
```
or, if a token was provided:
```text
Failed to get Vimeo thumbnails: Failed to get thumbnails. Tried oEmbed API and Vimeo API.
```

Raised when oEmbed produced no thumbnail URL, and either no `accessToken` was provided, or the API fallback also failed/returned no picture data.

---

## Troubleshooting

### "Video URL is required"

**Cause**

`videoUrl` was left empty or whitespace-only.

**Solution**

Provide a Vimeo video URL or a bare numeric video ID.

---

### "Invalid Vimeo URL. Could not extract video ID."

**Cause**

`videoUrl` doesn't match any recognized Vimeo URL pattern or a bare numeric ID.

**Solution**

Use a standard Vimeo video URL (`https://vimeo.com/<id>`) or just the numeric ID directly.

---

### "Failed to get thumbnails. Tried oEmbed API [and Vimeo API]."

**Cause**

Neither oEmbed nor (if attempted) the authenticated API returned any thumbnail data — commonly a private/restricted video, or the video doesn't exist.

**Solution**

If the video is private, provide a valid `accessToken` with access to view it, so the node can fall back to the authenticated Pictures API.

---

### All Three Sizes Are Identical URLs

**Cause**

This is expected when `method: "oembed"` — oEmbed only exposes a single thumbnail URL, which the node duplicates across `thumbnail_small`/`medium`/`large`. See the warning at the top of this document.

**Solution**

Provide a valid `accessToken` if genuinely different image sizes are required; the authenticated Pictures API returns real multiple sizes.

---

### `selectedThumbnail` Doesn't Match the Requested `thumbnailSize`

**Cause**

When `method: "api"`, the `thumbnails` map's keys come from Vimeo's own `size.type` values, which typically **don't** match the `thumbnail_small`/`thumbnail_medium`/`thumbnail_large` naming used by `thumbnailSize` — so the lookup misses and the node silently falls back to the first entry in the map, while still echoing the originally requested `selectedSize`.

**Solution**

Don't rely on `selectedSize` matching `selectedThumbnail`'s actual dimensions when `method: "api"`; instead, inspect the full `thumbnails` map and select the desired key/size directly downstream (e.g. in a `Function` node).

---

## Security

The node performs outbound HTTP requests to Vimeo's public oEmbed endpoint, and, as a fallback, the authenticated Vimeo API (`api.vimeo.com`).

`accessToken`, when provided, is sent as an `Authorization: Bearer <accessToken>` header only on the fallback API request.

---

## Notes

This node shares its overall two-tier lookup structure with [Vimeo Get Video](./vimeo-get-video.md), but the oEmbed path here is comparatively weaker: it only ever provides **one actual image**, duplicated under three size keys, whereas the authenticated API path provides genuinely different sizes.

The node does not:

- Guarantee real size variation between `thumbnail_small`/`medium`/`large` under `method: "oembed"`
- Guarantee that `thumbnailSize`'s requested key exists in `thumbnails` under `method: "api"` (silent fallback to the first available entry — see above)
- Retry either lookup attempt on transient failure
- Surface the specific underlying error from either failed attempt (both are swallowed)
- Support fetching thumbnails for multiple videos in one call

---

## Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-09-01 | Initial release |