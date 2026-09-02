---
node_id: "eleven-labs-text-to-speech-with-timestamps"
title: "Eleven Labs: Text to Speech (with Timestamps)"
description: "Convert text into speech and return character-level timing information using the ElevenLabs API."
category: "Generative AI & LLMs"
subcategory: "Multimodal AI"
version: "1.0.0"
language: "en"
last_updated: "2026-09-02"
author: "Fusion Team"
tags:
  - elevenlabs
  - text-to-speech
  - timestamps
  - alignment
  - audio
  - speech-synthesis
  - generative-ai
related_nodes:
  - eleven-labs-list-voices
  - eleven-labs-text-to-speech
  - eleven-labs-speech-to-speech
  - http-request
---

<!-- SECTION: header -->
# Eleven Labs: Text to Speech (with Timestamps)

> **Category:** Generative AI & LLMs | **Subcategory:** Multimodal AI | **Type:** Action Node

Convert text into generated speech and return character-level timing information for synchronizing audio with text.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Eleven Labs: Text to Speech (with Timestamps)** node generates audio from text and returns alignment data showing when each character starts and ends in the generated audio. This makes it useful for subtitles, highlighting text during playback, captions, karaoke-style interfaces, and accessible media experiences.

### Key Features

- **Speech Synthesis:** Convert text into audio using an ElevenLabs voice
- **Character Alignment:** Return start and end timing for each character
- **Normalized Alignment:** Provide timing based on normalized text when available
- **Voice Selection:** Use a voice ID from the ElevenLabs voice library
- **Voice Controls:** Configure stability, similarity, style, speaker boost, and speed
- **Deterministic Sampling:** Optionally provide a seed for more repeatable results
- **Secure Authentication:** Use an ElevenLabs API key through Fusion’s secret system
- **Error Routing:** Route authentication, validation, network, and API failures to the error output

### Use Cases

- Create synchronized captions and subtitles
- Highlight words or characters while audio plays
- Build accessible audio readers
- Produce karaoke and language-learning interfaces
- Align generated narration with video or animation
- Generate audio and timing data for media-processing pipelines

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
| `outputFormat` | `enum` | No | `mp3_44100_128` | Generated audio format and quality when supported by the node |
| `voiceSettings` | `object` | No | — | Optional stability, similarity, style, speaker boost, and speed settings |
| `applyTextNormalization` | `enum` | No | `auto` | Text normalization mode: `auto`, `on`, or `off` |
| `seed` | `number` | No | — | Seed used for best-effort repeatability; exact determinism is not guaranteed |
| `enableLogging` | `boolean` | No | `true` | Controls request logging where supported by the API |
| `optimizeStreamingLatency` | `number` | No | — | Optional latency optimization level; availability depends on the API version |

### API Request

The node uses the ElevenLabs text-to-speech endpoint with timestamps:

```text
POST https://api.elevenlabs.io/v1/text-to-speech/{voice_id}/with-timestamps
```

The API key is sent through the `xi-api-key` header. Text and voice options are sent in the request body.

### API Key Authentication

Store the API key in Fusion’s secret system and reference it dynamically:

```json
{
  "apiKey": "{{secrets.elevenLabsApiKey}}"
}
```

The example workflow contains only the placeholder `tap your Eleven Labs API key here`; no real key is included.

### Voice Settings

| Setting | Type | Description |
|---------|------|-------------|
| `stability` | `number` | Controls consistency and variation in delivery |
| `similarity_boost` | `number` | Controls similarity to the selected voice |
| `style` | `number` | Controls style exaggeration where supported |
| `use_speaker_boost` | `boolean` | Enables speaker similarity enhancement where supported |
| `speed` | `number` | Controls speaking speed; supported values depend on the API |

### Alignment Data

The response can include alignment arrays containing the original or normalized characters and matching start and end times in seconds. Array positions correspond to one another.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Input

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` | Optional dynamic input containing supported configuration overrides such as `text`, `voiceId`, or `voiceSettings` |

### Success Output

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Generated audio encoded for workflow handling together with alignment data |

### Success Output Example

```json
{
  "audio_base64": "<base64-audio-data>",
  "alignment": {
    "characters": ["H", "i"],
    "character_start_times_seconds": [0.0, 0.18],
    "character_end_times_seconds": [0.18, 0.34]
  },
  "normalized_alignment": {
    "characters": ["H", "i"],
    "character_start_times_seconds": [0.0, 0.18],
    "character_end_times_seconds": [0.18, 0.34]
  }
}
```

The runtime may expose the generated audio as a binary file or media object instead of a Base64 string.

### Error Output

Authentication failures, invalid voice IDs, invalid text, unsupported formats, rate limits, network failures, and ElevenLabs API errors are routed to the error output.

```json
{
  "success": false,
  "error": "ElevenLabs text-to-speech with timestamps request failed"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Generate Speech with Alignment Data

```json
{
  "apiKey": "{{secrets.elevenLabsApiKey}}",
  "voiceId": "voice-id-example",
  "text": "Hello, welcome to our service.",
  "applyTextNormalization": "auto",
  "enableLogging": true
}
```

### Generate Speech with Voice Settings

```json
{
  "apiKey": "{{secrets.elevenLabsApiKey}}",
  "voiceId": "voice-id-example",
  "text": "Your appointment is confirmed.",
  "voiceSettings": {
    "stability": 1,
    "similarity_boost": 1,
    "style": 0,
    "use_speaker_boost": true,
    "speed": 0.95
  },
  "seed": 1
}
```

### Synchronize Text with Audio

Use `alignment.character_start_times_seconds` and `alignment.character_end_times_seconds` to highlight characters or synchronize captions during playback.

```text
Text to Speech with Timestamps → Function → Captions, Player, or Animation
```

### Select a Voice Dynamically

Use **Eleven Labs: List Voices** to discover an accessible `voiceId` before generating audio.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Generate speech with character-level timestamps
```

### Common Patterns

- **Synchronized Captions:** Text to Speech with Timestamps → Function → Subtitle File
- **Voice Selection:** List Voices → Filter → Text to Speech with Timestamps
- **Audio Highlighting:** Text to Speech with Timestamps → Player or Frontend Data
- **Media Production:** Text Source → Speech with Timestamps → Audio and Video Pipeline
- **Accessibility:** Document → Speech with Timestamps → Audio Reader

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

**Cause:** The required `text` parameter is empty or was not passed through the incoming input.

**Solution:** Provide non-empty text in the node configuration or incoming workflow data.

#### Alignment arrays do not match

**Cause:** The consumer assumes a different text normalization or does not use the corresponding character array.

**Solution:** Use the characters from the same alignment object as the timing arrays, and distinguish `alignment` from `normalized_alignment`.

#### Audio format is unavailable

**Cause:** The selected format is unsupported or restricted by the ElevenLabs account plan.

**Solution:** Use the default output format or select a format supported by the current plan.

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
- **Eleven Labs: Text to Speech** - Generate speech without timing metadata
- **Eleven Labs: Speech to Speech** - Convert an existing recording to another voice
- [HTTP Request](../http-request/en.md) - Send generic HTTP requests

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-02 | Initial documentation |

<!-- /SECTION: changelog -->
