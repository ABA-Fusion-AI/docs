---
node_id: "eleven-labs-text-to-speech"
title: "Eleven Labs: Text to Speech"
description: "Convert text into natural-sounding speech using an ElevenLabs voice."
category: "Generative AI & LLMs"
subcategory: "Multimodal AI"
version: "1.0.0"
language: "en"
last_updated: "2026-09-01"
author: "Fusion Team"
tags:
  - elevenlabs
  - text-to-speech
  - speech-synthesis
  - audio
  - generative-ai
  - multimodal
related_nodes:
  - eleven-labs-list-voices
  - eleven-labs-speech-to-speech
  - http-request
---

<!-- SECTION: header -->
# Eleven Labs: Text to Speech

> **Category:** Generative AI & LLMs | **Subcategory:** Multimodal AI | **Type:** Action Node

Convert text into natural-sounding speech using a selected ElevenLabs voice and return the generated audio to the workflow.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Eleven Labs: Text to Speech** node sends text to ElevenLabs and generates an audio file using the selected voice. It supports output format selection, voice settings, text normalization, continuation text, and logging controls.

### Key Features

- **Speech Synthesis:** Convert written text into generated audio
- **Voice Selection:** Use an ElevenLabs voice ID
- **Audio Formats:** Select supported MP3, PCM, WAV, or other available output formats
- **Voice Controls:** Adjust stability, similarity, style, speaker boost, and speed
- **Text Normalization:** Control how numbers, dates, and symbols are pronounced
- **Natural Continuation:** Provide following text to improve continuity across generated sections
- **Secure Authentication:** Use an ElevenLabs API key through Fusion’s secret system
- **Error Routing:** Route authentication, validation, network, and API failures to the error output

### Use Cases

- Generate voiceovers for videos and presentations
- Create spoken notifications and announcements
- Convert articles, messages, or reports into audio
- Build multilingual or branded audio workflows
- Produce accessible audio versions of written content
- Generate audio after selecting a voice dynamically

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `apiKey` | `string` | Yes | — | ElevenLabs API key used to authenticate the request |
| `voiceId` | `string` | Yes | — | ElevenLabs voice ID used for speech generation |
| `text` | `string` | Yes | — | Text to convert into speech |
| `outputFormat` | `enum` | No | `mp3_44100_128` | Generated audio format and quality |
| `voiceSettings` | `object` | No | — | Optional voice controls such as stability, similarity, style, speaker boost, and speed |
| `applyTextNormalization` | `enum` | No | `auto` | Text normalization mode: `auto`, `on`, or `off` |
| `nextText` | `string` | No | — | Text that follows the current text, used to improve continuity |
| `enableLogging` | `boolean` | No | `true` | Controls request logging where supported by the ElevenLabs API |

### API Request

The node uses the ElevenLabs text-to-speech endpoint:

```text
POST https://api.elevenlabs.io/v1/text-to-speech/{voice_id}
```

The API key is sent through the `xi-api-key` request header. The text and voice settings are sent in the request body.

### API Key Authentication

Store the API key in Fusion’s secret system and reference it dynamically:

```json
{
  "apiKey": "{{secrets.elevenLabsApiKey}}"
}
```

Never commit a real ElevenLabs API key in a workflow file. The included example contains only the placeholder `tap your Eleven Labs API key here`.

### Voice Settings

The example supports the following voice settings:

| Setting | Type | Description |
|---------|------|-------------|
| `stability` | `number` | Controls consistency and variation in delivery |
| `similarity_boost` | `number` | Controls similarity to the selected voice |
| `style` | `number` | Controls style exaggeration where supported |
| `use_speaker_boost` | `boolean` | Enables speaker similarity enhancement where supported |
| `speed` | `number` | Controls speaking speed; supported values depend on the API |

### Output Format

Output formats use a codec, sample rate, and bitrate naming pattern such as:

```text
mp3_44100_128
```

Availability and quality limits depend on the ElevenLabs account plan and current API support.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Input

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` | Optional dynamic input containing supported configuration overrides such as `text`, `voiceId`, `outputFormat`, or `voiceSettings` |

### Success Output

| Output | Type | Description |
|--------|------|-------------|
| `success` | `binary` or `object` | Generated audio returned by ElevenLabs according to the runtime’s media-output handling |

### Request Example

```json
{
  "apiKey": "{{secrets.elevenLabsApiKey}}",
  "voiceId": "voice-id-example",
  "text": "Your appointment is confirmed.",
  "outputFormat": "mp3_44100_128",
  "voiceSettings": {
    "stability": 0.45,
    "similarity_boost": 1,
    "style": 0,
    "use_speaker_boost": true,
    "speed": 0.7
  },
  "enableLogging": true
}
```

### Error Output

Authentication failures, invalid voice IDs, invalid text, unsupported formats, rate limits, network failures, and ElevenLabs API errors are routed to the error output.

```json
{
  "success": false,
  "error": "ElevenLabs text-to-speech request failed"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Generate Standard Speech

```json
{
  "apiKey": "{{secrets.elevenLabsApiKey}}",
  "voiceId": "voice-id-example",
  "text": "Welcome to our service."
}
```

### Generate Speech with Voice Settings

```json
{
  "apiKey": "{{secrets.elevenLabsApiKey}}",
  "voiceId": "voice-id-example",
  "text": "Your appointment is confirmed for September 15, 2026.",
  "outputFormat": "mp3_44100_128",
  "voiceSettings": {
    "stability": 0.45,
    "similarity_boost": 1,
    "style": 0,
    "use_speaker_boost": true,
    "speed": 0.7
  },
  "applyTextNormalization": "on"
}
```

### Generate a Continued Segment

Use `nextText` when producing a segment that will be followed by another generated segment:

```json
{
  "text": "Thank you for contacting us.",
  "nextText": "We look forward to speaking with you soon."
}
```

### Select a Voice Dynamically

Use **Eleven Labs: List Voices** before this node to discover an accessible `voiceId`.

```text
List Voices → Select Voice → Text to Speech → Audio Storage
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: examples.workflows.json
title: Convert text into ElevenLabs speech
```

### Common Patterns

- **Basic Speech:** Manual Trigger → Text to Speech → Log or File Storage
- **Dynamic Voice:** List Voices → Filter → Text to Speech
- **Notification:** Event Trigger → Text to Speech → Audio Delivery
- **Content Pipeline:** Text Source → Text to Speech → Media Storage
- **Multilingual Audio:** Translate Text → Text to Speech → Distribution Channel

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### API key is missing

**Cause:** The `apiKey` parameter is empty or the referenced Fusion secret is not configured.

**Solution:** Add a valid ElevenLabs API key to the secret system and reference it with `{{secrets.elevenLabsApiKey}}`.

#### Unauthorized request

**Cause:** The API key is invalid, expired, revoked, or not sent through the `xi-api-key` header.

**Solution:** Verify the key and confirm that the node receives it through `apiKey`.

#### Voice ID is invalid

**Cause:** The selected voice does not exist or is not available to the authenticated account.

**Solution:** Use **Eleven Labs: List Voices** to retrieve an accessible voice ID.

#### Text is missing

**Cause:** The required `text` parameter is empty or was not passed from incoming workflow data.

**Solution:** Provide non-empty text in the node configuration or incoming input.

#### Audio format is unavailable

**Cause:** The selected output format is unsupported or restricted by the account plan.

**Solution:** Use the default `mp3_44100_128` format or select a format supported by the current ElevenLabs plan.

#### Text is pronounced incorrectly

**Cause:** Numbers, dates, symbols, or abbreviations may require text normalization.

**Solution:** Set `applyTextNormalization` to `on` or rewrite the text using words where precise pronunciation is important.

#### Rate limit or quota exceeded

**Cause:** The workflow exceeded the request limit or usage quota.

**Solution:** Reduce request frequency, add a delay, or review the ElevenLabs account plan and usage.

### Error Codes

| Error | Cause | Solution |
|-------|-------|----------|
| `HTTP 401` | Missing or invalid API key | Configure a valid secret-backed API key |
| `HTTP 403` | Account, voice, or plan restriction | Check account permissions and plan access |
| `HTTP 404` | Voice or endpoint not found | Verify the voice ID and endpoint |
| `HTTP 422` | Invalid text, settings, or output format | Check required fields and supported values |
| `HTTP 429` | Rate limit or quota exceeded | Reduce request frequency and retry later |
| `HTTP 5xx` | ElevenLabs service failure | Retry after a short delay |
| `Network error` | Connection failure | Check connectivity and API availability |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **Eleven Labs: List Voices** - Discover available voice IDs
- **Eleven Labs: Speech to Speech** - Convert an existing recording to another voice
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
