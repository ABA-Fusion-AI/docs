---
node_id: "vimeo-get-video"
title: "Vimeo Get Video"
description: "Get Vimeo video details. Tries oEmbed API first (no auth), then falls back to Vimeo API if token provided."
category: "Media / Video"
version: "1.0.0"
language: "en"
last_updated: "2026-09-01"
author: "Fusion Team"
tags:

- vimeo
- video
- oembed
- media
- metadata
- api

related_nodes:
- function
- if
- http-request

---

# Vimeo Get Video

> **Category:** media-nodes | **Type:** Action Node

Get **Vimeo video details** from a video URL, trying the **oEmbed API first (no authentication required)**, and falling back to the **authenticated Vimeo API** if an access token is provided and oEmbed doesn't succeed.

The **Vimeo Get Video** node extracts a video ID from the given URL, attempts an unauthenticated oEmbed lookup, and only calls the full Vimeo API (which requires a Bearer token) as a fallback — useful for retrieving basic metadata for public videos without needing Vimeo API credentials at all.

### Supported Features

- Video ID extraction from several Vimeo URL formats (including channel/group/album URLs, and a bare numeric ID)
- Primary lookup via Vimeo's public oEmbed API — no authentication needed
- Automatic fallback to the authenticated Vimeo API (`api.vimeo.com`) if oEmbed fails and an `accessToken` is provided
- oEmbed response reshaped into a Vimeo-API-like `video` object for output consistency
- Reports which method (`oembed` or `api`) actually produced the result

---

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `videoUrl` | `string` | ✅ Yes | — | Vimeo video URL (or a bare numeric video ID). Must be non-empty. |
| `accessToken` | `string` | ❌ No | — | Vimeo API access token, used only as a fallback if oEmbed fails. |

---

## Video ID Extraction

`videoUrl` is matched against two patterns, in order:

1. `vimeo\.com\/(?:channels\/[^\/]+\/|groups\/[^\/]+\/videos\/|album\/\d+\/video\/|)(\d+)` — matches standard Vimeo URLs, including channel URLs (`vimeo.com/channels/staffpicks/123456789`), group URLs (`vimeo.com/groups/name/videos/123456789`), and album URLs (`vimeo.com/album/456/video/123456789`), as well as plain `vimeo.com/123456789`.
2. `^(\d+)$` — matches a bare numeric string with nothing else, treating it directly as a video ID.

If neither pattern matches, the node throws `"Invalid Vimeo URL. Could not extract video ID."`.

---

## Lookup Strategy

1. **Try oEmbed** (`https://vimeo.com/api/oembed.json?url=<videoUrl>`) — no authentication. If this succeeds (`response.ok`), its response is reshaped into a Vimeo-API-like structure and returned immediately, with `method: "oembed"`.
2. **If oEmbed didn't produce a result** (failed request, threw, or returned a non-OK status) **and `accessToken` is set and non-blank** — try the authenticated Vimeo API (`https://api.vimeo.com/videos/{videoId}`) with the token as a Bearer header. If this succeeds, the **raw** Vimeo API response is returned as `video`, with `method: "api"`.
3. **If neither attempt produced a result** — the node throws, with a message noting whether the API fallback was even attempted (i.e. whether `accessToken` was provided).

Both attempts are wrapped in internal try/catch blocks that **silently swallow errors** (no `console.error`, no partial error detail retained) — only the final "both attempts failed" state is surfaced.

---

## oEmbed-to-API Field Mapping

When the oEmbed path succeeds, its fields are mapped into a Vimeo-API-shaped object as follows:

| oEmbed Field | Mapped To | Notes |
| ------------- | ---------- | ----- |
| (extracted from URL) | `video.id` | Not from oEmbed itself — taken from the URL-parsed video ID. |
| `title` | `video.name` | Falls back to `""` if missing. |
| `description` | `video.description` | Falls back to `""`. |
| `duration` | `video.duration` | Falls back to `null`. |
| `width` / `height` | `video.width` / `video.height` | Fall back to `null`. |
| `html` | `video.embed.html` | Falls back to `""`. |
| `thumbnail_width` / `thumbnail_height` / `thumbnail_url` | `video.pictures.sizes[0]` | Width/height fall back to `640`/`360`; `link` falls back to `""`. |
| `video_id` | `video.link` | Falls back to the original `videoUrl` if `oembedData.video_id` is missing. |
| `author_name` / `author_url` | `video.user.name` / `video.user.link` | Fall back to `""`. |
| (not provided by oEmbed) | `video.stats.plays` | Always `null` — oEmbed doesn't expose play counts. |

This means the **oEmbed-derived output is intentionally shaped to resemble the real Vimeo API response**, but it is a subset — several fields available from the full API (e.g. `stats.plays`, privacy settings, tags, full user profile) are simply not obtainable via oEmbed and appear as `null`/`""`/absent.

---

## Inputs & Outputs

### Inputs

The node does not require workflow input. All configuration is provided through the node configuration.

### Outputs

| Output | Type | Description |
| ------ | ---- | ----------- |
| `success` | `boolean` | Always `true` if a result was obtained (the node throws instead of returning `success: false`). |
| `video` | `object` | Video details — shape depends on `method` (see [oEmbed-to-API Field Mapping](#oembed-to-api-field-mapping) for the `oembed` shape; raw Vimeo API response for the `api` shape). |
| `method` | `string` | `"oembed"` or `"api"` — which lookup path actually produced the result. |

**Note:** the `video` object's shape is **not consistent** between `method: "oembed"` and `method: "api"` — the oEmbed path returns a specific reshaped subset of fields, while the API path returns Vimeo's full, differently-structured video resource. Downstream nodes should check `method` before assuming a particular field is present.

---

## Output Example

### `method: "oembed"`

```json
{
  "success": true,
  "video": {
    "id": "123456789",
    "name": "My Vimeo Video",
    "description": "A short demo video.",
    "duration": 142,
    "width": 1920,
    "height": 1080,
    "embed": { "html": "<iframe src=\"https://player.vimeo.com/video/123456789\" ...></iframe>" },
    "pictures": {
      "sizes": [
        { "width": 640, "height": 360, "link": "https://i.vimeocdn.com/video/thumb.jpg" }
      ]
    },
    "link": "https://vimeo.com/123456789",
    "user": { "name": "Jane Doe", "link": "https://vimeo.com/janedoe" },
    "stats": { "plays": null }
  },
  "method": "oembed"
}
```

### `method: "api"` (abbreviated, raw Vimeo API shape)

```json
{
  "success": true,
  "video": {
    "uri": "/videos/123456789",
    "name": "My Vimeo Video",
    "description": "A short demo video.",
    "duration": 142,
    "width": 1920,
    "height": 1080,
    "stats": { "plays": 4821 },
    "privacy": { "view": "anybody" }
  },
  "method": "api"
}
```

---

## Configuration Examples

### Public Video, No Token (oEmbed Only)

```json
{
  "videoUrl": "https://vimeo.com/123456789"
}
```

### With Fallback Token

```json
{
  "videoUrl": "https://vimeo.com/123456789",
  "accessToken": "your-vimeo-access-token"
}
```

### Bare Video ID

```json
{
  "videoUrl": "123456789"
}
```

### Channel URL

```json
{
  "videoUrl": "https://vimeo.com/channels/staffpicks/123456789"
}
```

---

## Workflow Integration

### Sample Workflow: Get Video → Function

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
      "id": "format-video-card",
      "type": "function"
    }
  ]
}
```

### Sample Workflow: Get Video (with token) → If → Notification

```json
{
  "nodes": [
    {
      "id": "vimeo-get-video",
      "type": "vimeo-get-video",
      "config": {
        "videoUrl": "https://vimeo.com/123456789",
        "accessToken": "your-vimeo-access-token"
      }
    },
    {
      "id": "check-play-count",
      "type": "if"
    },
    {
      "id": "notify-milestone",
      "type": "notification"
    }
  ]
}
```

### Common Patterns

- Vimeo Get Video → Function (check `method`) → format accordingly — handle both oEmbed and API response shapes
- Vimeo Get Video (with token) → If (stats/privacy check) → Notification — requires the API fallback for fields oEmbed doesn't provide
- Function (extract video URL from content) → Vimeo Get Video → Database — video metadata archiving

---

## Error Handling

### Missing Video URL

```text
Video URL is required
```

Raised when `videoUrl` is empty or whitespace-only.

### Invalid URL

```text
Failed to get Vimeo video: Invalid Vimeo URL. Could not extract video ID.
```

Raised when neither ID-extraction pattern matches `videoUrl`.

### Both Lookups Failed

```text
Failed to get Vimeo video: Failed to get video details. Tried oEmbed API.
```
or, if a token was provided:
```text
Failed to get Vimeo video: Failed to get video details. Tried oEmbed API and Vimeo API.
```

Raised when oEmbed failed/returned non-OK, and either no `accessToken` was provided, or the API fallback also failed/returned non-OK.

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

`videoUrl` doesn't match any of the recognized Vimeo URL patterns (or a bare numeric ID) — e.g. it's a non-Vimeo URL, or an unusual Vimeo URL format not covered by the regex patterns.

**Solution**

Use a standard Vimeo video URL (`https://vimeo.com/<id>`) or just the numeric ID directly.

---

### "Failed to get video details. Tried oEmbed API."

**Cause**

The oEmbed lookup failed or returned a non-OK status (commonly: the video is private, unlisted with embed restrictions, or does not exist), and **no `accessToken` was provided** to attempt the authenticated fallback.

**Solution**

If the video is private or has restricted embedding, provide a valid `accessToken` with appropriate access so the node can fall back to the authenticated Vimeo API.

---

### "Failed to get video details. Tried oEmbed API and Vimeo API."

**Cause**

Both the oEmbed lookup and the authenticated API fallback failed — the video may not exist, the `accessToken` may lack permission to view it, or the token itself may be invalid/expired.

**Solution**

Verify the video ID/URL is correct, and that the `accessToken` is valid and has appropriate scope/permissions for the video in question.

---

### `video` Shape Looks Different Between Two Calls

**Cause**

This is expected — `method` determines the shape: `"oembed"` returns the reshaped subset described in [oEmbed-to-API Field Mapping](#oembed-to-api-field-mapping), while `"api"` returns Vimeo's native, differently-structured video resource.

**Solution**

Always branch on the `method` field downstream before accessing shape-specific fields (e.g. `stats.plays` is reliably numeric only under `method: "api"`).

---

### No Specific Error Detail on Why oEmbed or API Failed

**Cause**

Both lookup attempts catch and silently discard their internal errors (no logging, no error message retention) — only the generic "Tried oEmbed API..." message is ever surfaced, regardless of the underlying HTTP status or network error.

**Solution**

If more detail is needed to diagnose a failure, use an [HTTP Request](./http-request.md) node directly against the oEmbed or Vimeo API endpoint to see the actual response/status.

---

## Security

The node performs outbound HTTP requests to Vimeo's public oEmbed endpoint (`vimeo.com/api/oembed.json`), and, as a fallback, the authenticated Vimeo API (`api.vimeo.com`).

`accessToken`, when provided, is sent as an `Authorization: Bearer <accessToken>` header only on the fallback API request — never on the oEmbed request, which requires no authentication.

---

## Notes

The node is designed to **avoid requiring Vimeo API credentials** for the common case of retrieving basic metadata for a public video — the API token is genuinely optional and only used when oEmbed doesn't succeed.

The node does not:

- Guarantee a consistent output shape across `method` values (see above)
- Retry either lookup attempt on transient failure
- Surface the specific underlying error from either failed attempt (both are swallowed)
- Support fetching multiple videos in one call
- Support any operation beyond fetching video details (no listing, uploading, or editing)

---

## Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-09-01 | Initial release |