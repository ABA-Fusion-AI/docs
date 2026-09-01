---
node_id: "vimeo-get-transcript"
title: "Vimeo Get Transcript"
description: "Get Vimeo video transcript/captions. Requires access token for Vimeo API (texttracks endpoint)."
category: "Media / Video"
version: "1.0.0"
language: "en"
last_updated: "2026-09-01"
author: "Fusion Team"
tags:

- vimeo
- transcript
- captions
- subtitles
- vtt
- srt
- media
- api

related_nodes:
- vimeo-get-video
- function
- if

---

# Vimeo Get Transcript

> **Category:** media-nodes | **Type:** Action Node

Get a **Vimeo video's transcript or captions** (text tracks) in a chosen output format, via the authenticated Vimeo API's `texttracks` endpoint.

The **Vimeo Get Transcript** node lists a video's available caption/transcript tracks, selects the best match for a requested language, downloads the raw VTT content, and converts it into plain text, SRT, VTT, or a JSON wrapper.

## ⚠️ Important: Access Token Is Mandatory (Unlike Vimeo Get Video)

Unlike [Vimeo Get Video](./vimeo-get-video.md), which can work without any credentials via oEmbed, **this node always requires `accessToken`** — the `texttracks` endpoint is only available through the authenticated Vimeo API, with no public/oEmbed equivalent. The node throws immediately if `accessToken` is missing, before even attempting a request.

### Supported Features

- Video ID extraction from several Vimeo URL formats (same patterns as [Vimeo Get Video](./vimeo-get-video.md))
- Lists all available text tracks (transcripts/captions) for a video
- Language matching: exact match, or a base-language fallback (e.g. requesting `en` matches an available `en-US` track)
- Falls back to the first available track if no language match is found
- Multiple output formats: plain text, SRT, raw VTT, or a JSON wrapper around the raw VTT
- Always returns the raw, unconverted VTT text (`transcriptRaw`) alongside the requested format
- Graceful "no transcript available" response (not thrown) when a video has no tracks or no matching language

---

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `videoUrl` | `string` | ✅ Yes | — | Vimeo video URL (or a bare numeric video ID). Must be non-empty. |
| `accessToken` | `string` | ✅ Yes | — | Vimeo API access token. Always required for this node (see warning above). |
| `language` | `string` | ❌ No | `"en"` | Preferred transcript language (e.g. `"en"`, `"fr"`, `"en-US"`). |
| `outputFormat` | `enum` | ❌ No | `"text"` | Output format: `vtt`, `srt`, `txt`, `text`, or `json`. Note: `txt` and `text` behave identically. |

---

## Video ID Extraction

Identical logic to [Vimeo Get Video](./vimeo-get-video.md#video-id-extraction): matches standard Vimeo URLs (including channel/group/album variants) or a bare numeric ID. Throws `"Invalid Vimeo URL. Could not extract video ID."` if neither pattern matches.

---

## Lookup and Download Flow

1. **List text tracks**: `GET https://api.vimeo.com/videos/{videoId}/texttracks`, authenticated with `accessToken`.
2. **If no tracks exist** — returns (does not throw) `{ success: false, message: "No transcripts/captions available for this video", availableLanguages: null }`.
3. **Select a track** for `language`, in this priority order:
   - Exact match on `track.language === language`.
   - Base-language match: `track.language` starts with the base part of `language` (i.e. `language.split("-")[0]`) — so requesting `"en"` matches an available `"en-US"` or `"en-GB"` track, and requesting `"en-US"` still matches a plain `"en"` track via its own base check.
   - **Fallback**: if no match at all, the node uses `availableTracks[0]` — the **first track in whatever order the API returned them**, not necessarily an English or "default" track.
4. **If, after all that, no track was selected** (only possible if `availableTracks` were somehow empty at this point, which the step-2 check should already have caught) — returns `{ success: false, message: "No transcript available for language: <language>", availableLanguages: [...] }`.
5. **Download** the selected track's `link` (falling back to `uri` if `link` is empty) as raw text — this is VTT-formatted caption text.
6. **Convert** the raw VTT into the requested `outputFormat`.

---

## Output Format Conversion

| `outputFormat` | Conversion Applied |
| ----------------- | -------------------- |
| `"vtt"` (or any unrecognized value) | Raw VTT text returned as-is. |
| `"txt"` / `"text"` | Converted via `vttToText`: strips `WEBVTT` header, cue-index lines, and timestamp lines, joins remaining text lines with spaces, and collapses whitespace. |
| `"srt"` | Converted via `vttToSrt`: re-parses VTT timestamps (`HH:MM:SS.mmm --> HH:MM:SS.mmm`) into SRT format (`HH:MM:SS,mmm --> HH:MM:SS,mmm`) with sequential cue numbers. |
| `"json"` | Wrapped as `{ raw: "<raw VTT text>" }` — **not** parsed into structured cue objects, despite the name suggesting otherwise. |

Regardless of `outputFormat`, the **unconverted raw VTT text** is always additionally available in the `transcriptRaw` output field.

---

## Inputs & Outputs

### Inputs

The node does not require workflow input. All configuration is provided through the node configuration.

### Outputs — Success

| Output | Type | Description |
| ------ | ---- | ----------- |
| `success` | `boolean` | `true`. |
| `videoUrl` | `string` | The input video URL. |
| `videoId` | `string` | Extracted numeric video ID. |
| `language` | `string` | The actual language code of the track that was selected (may differ from the requested `language` if a fallback occurred). |
| `format` | `string` | The `outputFormat` that was used. |
| `availableLanguages` | `string[]` | Language codes of all tracks available for the video. |
| `transcript` | `string \| object` | The transcript in the requested `outputFormat` (a string for `vtt`/`txt`/`text`/`srt`, an object `{ raw }` for `json`). |
| `transcriptRaw` | `string` | The raw, unconverted VTT text, always present regardless of `outputFormat`. |

### Outputs — No Transcript Available (Not Thrown)

| Output | Type | Description |
| ------ | ---- | ----------- |
| `success` | `boolean` | `false`. |
| `videoUrl` | `string` | The input video URL. |
| `videoId` | `string` | Extracted numeric video ID. |
| `message` | `string` | `"No transcripts/captions available for this video"` or `"No transcript available for language: <language>"`. |
| `availableLanguages` | `string[] \| null` | `null` if the video has no tracks at all; otherwise the list of available language codes. |

---

## Output Example

### Success (`outputFormat: "text"`)

```json
{
  "success": true,
  "videoUrl": "https://vimeo.com/123456789",
  "videoId": "123456789",
  "language": "en-US",
  "format": "text",
  "availableLanguages": ["en-US", "fr"],
  "transcript": "Welcome to this demo video. Today we'll cover three main topics...",
  "transcriptRaw": "WEBVTT\n\n1\n00:00:00.000 --> 00:00:03.500\nWelcome to this demo video.\n\n2\n00:00:03.500 --> 00:00:07.000\nToday we'll cover three main topics...\n"
}
```

### Success (`outputFormat: "srt"`)

```json
{
  "success": true,
  "videoUrl": "https://vimeo.com/123456789",
  "videoId": "123456789",
  "language": "en-US",
  "format": "srt",
  "availableLanguages": ["en-US", "fr"],
  "transcript": "1\n00:00:00,000 --> 00:00:03,500\nWelcome to this demo video.\n\n2\n00:00:03,500 --> 00:00:07,000\nToday we'll cover three main topics...\n",
  "transcriptRaw": "WEBVTT\n\n1\n00:00:00.000 --> 00:00:03.500\n..."
}
```

### No Transcripts Available

```json
{
  "success": false,
  "videoUrl": "https://vimeo.com/987654321",
  "videoId": "987654321",
  "message": "No transcripts/captions available for this video",
  "availableLanguages": null
}
```

---

## Configuration Examples

### Default (English Text)

```json
{
  "videoUrl": "https://vimeo.com/123456789",
  "accessToken": "your-vimeo-access-token"
}
```

### French SRT Subtitles

```json
{
  "videoUrl": "https://vimeo.com/123456789",
  "accessToken": "your-vimeo-access-token",
  "language": "fr",
  "outputFormat": "srt"
}
```

### Raw VTT

```json
{
  "videoUrl": "https://vimeo.com/123456789",
  "accessToken": "your-vimeo-access-token",
  "outputFormat": "vtt"
}
```

---

## Workflow Integration

### Sample Workflow: Get Transcript → LLM (summarize)

```json
{
  "nodes": [
    {
      "id": "vimeo-transcript",
      "type": "vimeo-get-transcript",
      "config": {
        "videoUrl": "https://vimeo.com/123456789",
        "accessToken": "your-vimeo-access-token"
      }
    },
    {
      "id": "summarize-transcript",
      "type": "llm"
    }
  ]
}
```

### Sample Workflow: Get Video → Get Transcript (chained)

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
      "id": "vimeo-transcript",
      "type": "vimeo-get-transcript",
      "config": {
        "videoUrl": "https://vimeo.com/123456789",
        "accessToken": "your-vimeo-access-token",
        "outputFormat": "srt"
      }
    },
    {
      "id": "build-video-package",
      "type": "function"
    }
  ]
}
```

### Sample Workflow: Get Transcript → If (no transcript) → Notification

```json
{
  "nodes": [
    {
      "id": "vimeo-transcript",
      "type": "vimeo-get-transcript",
      "config": {
        "videoUrl": "https://vimeo.com/123456789",
        "accessToken": "your-vimeo-access-token"
      }
    },
    {
      "id": "check-transcript-success",
      "type": "if"
    },
    {
      "id": "notify-missing-captions",
      "type": "notification"
    }
  ]
}
```

### Common Patterns

- Vimeo Get Transcript → LLM — video content summarization or Q&A
- Vimeo Get Transcript (`srt`) → Function → File export — subtitle file generation
- Vimeo Get Transcript → If (`success`) → Notification — flag videos missing captions

---

## Error Handling

### Missing Video URL

```text
Video URL is required
```

Raised when `videoUrl` is empty or whitespace-only.

### Missing Access Token

```text
Vimeo access token is required for transcripts. Get one from https://developer.vimeo.com/apps
```

Raised unconditionally when `accessToken` is empty or whitespace-only — checked before any URL parsing or request.

### Invalid URL

```text
Failed to get Vimeo transcript: Invalid Vimeo URL. Could not extract video ID.
```

### Text Tracks List API Error

```text
Failed to get Vimeo transcript: Vimeo API error: <error message or statusText>
```

Raised when the `texttracks` list request returns a non-OK status.

### Transcript Download Failure

```text
Failed to get Vimeo transcript: Failed to download transcript: <status> <statusText>
```

Raised when downloading the selected track's content fails.

### Missing Download URL

```text
Failed to get Vimeo transcript: No download URL found for transcript
```

Raised in the unlikely case that the selected track has neither a `link` nor a `uri` field.

---

## Troubleshooting

### "Vimeo access token is required for transcripts. ..."

**Cause**

`accessToken` was left empty — this node has no unauthenticated path, unlike [Vimeo Get Video](./vimeo-get-video.md).

**Solution**

Obtain a Vimeo API access token from https://developer.vimeo.com/apps with appropriate scope to read the video's text tracks.

---

### `success: false, message: "No transcripts/captions available for this video"`

**Cause**

The video genuinely has no captions or transcripts uploaded on Vimeo.

**Solution**

Upload captions/subtitles to the video via the Vimeo dashboard or API first, or accept that no transcript exists for this video.

---

### `success: false, message: "No transcript available for language: <language>"`

**Cause**

This case is effectively unreachable in the current implementation — since step 3 always falls back to `availableTracks[0]` when no language match is found, `requestedTrack` will always be truthy as long as `availableTracks` is non-empty (which was already checked in step 2). This message path exists as defensive code but should not normally trigger.

**Solution**

If seen unexpectedly, treat it as a rare edge case; check `availableLanguages` in the response and adjust `language` to match one of the listed codes.

---

### Wrong Language Track Returned

**Cause**

No exact or base-language match was found for the requested `language`, so the node silently fell back to `availableTracks[0]` — the **first track in API response order**, which is not guaranteed to be a sensible default (e.g. not necessarily English).

**Solution**

Check the `language` field in the response to see which track was actually used, and compare against `availableLanguages` to pick a value that will actually match.

---

### `transcript` is `{ "raw": "..." }` Instead of Structured Cues

**Cause**

`outputFormat: "json"` does **not** parse the VTT into structured `{ start, end, text }` cue objects — it only wraps the raw VTT string. This may be surprising given the format name.

**Solution**

If structured per-cue data is needed, use `outputFormat: "srt"` or `"vtt"` and parse the result downstream with a dedicated `Function` node, or parse `transcriptRaw` directly.

---

### "Vimeo API error: ..." on the Text Tracks List

**Cause**

The `accessToken` may lack the required scope to view text tracks, or the video ID is invalid/inaccessible to the authenticated identity.

**Solution**

Verify the token has appropriate read scope and that the authenticated account can access the video (private videos require the token's identity to have viewing rights).

---

## Security

The node authenticates both the text-tracks listing request and the transcript download request with `accessToken`, sent as an `Authorization: Bearer <accessToken>` header.

Unlike [Vimeo Get Video](./vimeo-get-video.md), there is no unauthenticated fallback path — a valid token with transcript-read access is always required.

---

## Notes

The `outputFormat: "json"` option is a thin wrapper (`{ raw: "<VTT text>" }`), not a structured cue-by-cue JSON representation — despite what the name might suggest, use `srt`/`vtt` plus downstream parsing if structured cue data is needed.

The node does not:

- Support fetching transcripts for multiple languages in a single call
- Guarantee the fallback track (when no language matches) is any particular language — it's simply the first one returned by the API
- Parse VTT into structured cue objects for the `json` format
- Cache transcript results between calls
- Support uploading or editing captions (read/download only)

---

## Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-09-01 | Initial release |