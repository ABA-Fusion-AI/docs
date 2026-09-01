---
node_id: "eleven-labs-speech-to-speech"
title: "Eleven Labs: Speech to Speech"
description: "Convert an audio recording from one voice to another using the ElevenLabs voice-conversion API."
category: "Generative AI & LLMs "
subcategory: "Multimodal AI"
version: "1.0.0"
language: "en"
last_updated: "2026-09-01"
author: "Fusion Team"
tags:
  - elevenlabs
  - speech-to-speech
  - voice-conversion
  - audio
  - generative-ai
  - multimodal
related_nodes:
  - eleven-labs-list-voices
  - eleven-labs-text-to-speech
  - eleven-labs-speech-to-text
  - http-request
---

<!-- SECTION: header -->
# Eleven Labs: Speech to Speech

> **Category:** Generative AI & LLMs | **Subcategory:** Multimodal AI | **Type:** Action Node

Convert an audio recording from one voice to another while preserving the original delivery, timing, and expressive qualities where supported by the ElevenLabs API.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Eleven Labs: Speech to Speech** node sends an audio file to ElevenLabs and generates a new audio file using the selected target voice. It is useful for voice conversion, localization, character voices, and other audio-production workflows.

### Key Features

- **Voice Conversion:** Transform source speech into a selected target voice
- **Audio URL Input:** Read the source audio from a URL
- **Voice Selection:** Use a voice ID from the ElevenLabs voice library
- **Expressive Delivery:** Preserve timing and delivery characteristics where supported
- **Secure Authentication:** Use an ElevenLabs API key through Fusion’s secret system
- **Workflow Integration:** Pass generated audio to storage, delivery, or downstream media nodes
- **Error Routing:** Route authentication, validation, network, and API failures to the error output

### Use Cases

- Convert narration into a different voice
- Create localized or branded audio content
- Apply character voices to existing recordings
- Build voice-production and media-processing workflows
- Combine voice discovery, conversion, and audio storage steps

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `apiKey` | `string` | Yes | — | ElevenLabs API key used to authenticate the request |
| `voiceId` | `string` | Yes | — | Target ElevenLabs voice ID |
| `audioUrl` | `url` | Yes | — | Publicly accessible source audio URL |

### API Request

The node uses the ElevenLabs voice-conversion endpoint:

```text
POST https://api.elevenlabs.io/v1/speech-to-speech/{voice_id}
```

The request sends the source audio as multipart form data and authenticates using the `xi-api-key` header.

### API Key Authentication

Store the API key in Fusion’s secret system and reference it dynamically:

```json
{
  "apiKey": "{{secrets.elevenLabsApiKey}}"
}
```

Never commit a real API key in a workflow file. The existing example workflow contains a literal key-shaped value and should be treated as exposed; revoke or rotate it and replace it with a secret reference.

### Voice ID

Use the **Eleven Labs: List Voices** node to discover available voice IDs. The selected voice must be available to the authenticated account and compatible with the configured ElevenLabs service.

### Audio Requirements

The source URL must be reachable by the workflow runtime and point to a supported audio file. Ensure the file is accessible without an interactive login and that its format and size comply with the ElevenLabs account and endpoint limits.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Input

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` | Optional dynamic input containing `apiKey`, `voiceId`, and `audioUrl` overrides |

### Success Output

| Output | Type | Description |
|--------|------|-------------|
| `success` | `binary` or `object` | Converted audio returned by ElevenLabs, according to the runtime’s media-output handling |

### Request Example

```json
{
  "apiKey": "{{secrets.elevenLabsApiKey}}",
  "voiceId": "voice-id-example",
  "audioUrl": "https://example.com/source-audio.wav"
}
```

### Error Output

Authentication failures, invalid voice IDs, inaccessible audio URLs, unsupported media, rate limits, network failures, and ElevenLabs API errors are routed to the error output.

```json
{
  "success": false,
  "error": "ElevenLabs speech-to-speech request failed"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Convert Audio to a Selected Voice

```json
{
  "apiKey": "{{secrets.elevenLabsApiKey}}",
  "voiceId": "voice-id-example",
  "audioUrl": "https://example.com/source-audio.wav"
}
```

### Select a Voice Dynamically

Connect **Eleven Labs: List Voices** before this node, select a `voice_id`, and pass that value into `voiceId`.

```text
List Voices → Select Voice → Speech to Speech
```

### Process the Converted Audio

Send the success output to a file, object-storage, media, or notification node for further processing or delivery.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Convert source audio to an ElevenLabs voice
```

### Common Patterns

- **Basic Conversion:** Manual Trigger → Speech to Speech → Log
- **Voice Selection:** List Voices → Function or Filter → Speech to Speech
- **Media Pipeline:** Speech to Speech → File Storage → Notification
- **Localization:** Audio Input → Speech to Speech → Distribution Channel
- **Quality Review:** Speech to Speech → Log or Approval Step

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### API key is missing

**Cause:** The `apiKey` parameter is empty or the referenced Fusion secret is not configured.

**Solution:** Add a valid ElevenLabs API key to the secret system and reference it with `{{secrets.elevenLabsApiKey}}`.

#### Unauthorized request

**Cause:** The API key is invalid, expired, revoked, or not sent as the `xi-api-key` header.

**Solution:** Verify the key and confirm that the node receives it through `apiKey`.

#### Voice ID is invalid

**Cause:** The target voice does not exist or is not available to the authenticated account.

**Solution:** Use **Eleven Labs: List Voices** to retrieve an accessible voice ID.

#### Audio URL cannot be downloaded

**Cause:** The URL is private, expired, blocked, unreachable, or requires authentication that the node cannot provide.

**Solution:** Use a stable, publicly reachable audio URL and verify that the workflow runtime can access it.

#### Unsupported audio input

**Cause:** The source file format, size, duration, or encoding is not accepted by the endpoint or account plan.

**Solution:** Convert the source audio to a supported format and check the current ElevenLabs limits.

#### Rate limit exceeded

**Cause:** The workflow exceeded the request limit or account quota.

**Solution:** Reduce request frequency, add a delay, or review the ElevenLabs account plan and usage.

### Error Codes

| Error | Cause | Solution |
|-------|-------|----------|
| `HTTP 401` | Missing or invalid API key | Configure a valid secret-backed API key |
| `HTTP 403` | Account or voice-access restriction | Check account permissions and voice availability |
| `HTTP 404` | Voice or endpoint not found | Verify the voice ID and API endpoint |
| `HTTP 422` | Invalid request or unsupported audio | Check the voice ID, URL, and audio format |
| `HTTP 429` | Rate limit or quota exceeded | Reduce request frequency and retry later |
| `HTTP 5xx` | ElevenLabs service failure | Retry after a short delay |
| `Network error` | Connection or download failure | Check connectivity and source URL access |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **Eleven Labs: List Voices** - Discover available target voice IDs
- **Eleven Labs: Text to Speech** - Generate speech from text using a selected voice
- **Eleven Labs: Speech to Text** - Transcribe audio into text
- [HTTP Request](../http-request/en.md) - Send generic HTTP requests

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-01 | Initial documentation |

<!-- /SECTION: changelog -->
