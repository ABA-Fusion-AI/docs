---
node_id: "deepgram"
title: "Deepgram"
description: "Convert text to speech using Deepgram API (STT/TTS)."
category: "peer-only"
subcategory: "integrations"
version: "1.0.0"
language: "en"
last_updated: "2026-08-11"
author: "Fusion Team"
tags:
  - deepgram
  - tts
  - stt
  - text-to-speech
  - audio
  - ai
  - speech
  - Peer-only
  - action
related_nodes:
  - function
  - webhook-trigger
  - http-request
---

<!-- SECTION: overview -->
# Deepgram

> **Category:** Peer-only  | **Type:** Action Node

Convert written text into lifelike, high-quality spoken audio using Deepgram's Text-to-Speech (TTS) API.

The **Deepgram** node enables workflows to synthesize natural voice audio dynamically from text inputs. It supports configurable voice models (such as `aura-asteria-en`), custom encoding types (MP3, Linear16 PCM, AAC, Opus), audio containers (WAV, OGG, MP3), and custom voice/prompt parameters.

### Use Cases

- Generate automated voice responses for customer support bots and IVR systems
- Convert dynamic email or notification text into spoken audio announcements
- Produce audio files for podcasts, articles, or accessibility screen readers
- Generate voice clips in specific audio formats (e.g. WAV linear16 for telecom, MP3 for web streaming)
- Dynamically speak translated text in multilingual AI assistant workflows

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `apiKey` | `string` | ✅ Yes | `""` | Deepgram API Key for authentication |
| `text` | `string` | ✅ Yes | `""` | Input text string to synthesize into speech |
| `model` | `string` | ❌ No | `"aura-asteria-en"` | Deepgram voice model identifier |
| `voice` | `string` | ❌ No | `""` | Custom voice prompt or voice style identifier |
| `encoding` | `string` | ❌ No | `"mp3"` | Audio encoding output format (`mp3`, `linear16`, `opus`, `aac`, `flac`, `alaw`, `mulaw`) |
| `container` | `string` | ❌ No | `"mp3"` | Audio container format (`mp3`, `wav`, `ogg`, `none`) |

---

### Deepgram Voice Models

Deepgram provides several high-quality Aura voice models optimized for low-latency conversational and narration tasks:

| Model ID | Description |
|----------|-------------|
| `aura-asteria-en` | English (US) Female voice – Warm, clear, conversational (Default) |
| `aura-luna-en` | English (US) Female voice – Friendly, energetic |
| `aura-stella-en` | English (US) Female voice – Professional, authoritative |
| `aura-athena-en` | English (UK) Female voice – Calm, polished |
| `aura-hera-en` | English (US) Female voice – Soft, articulate |
| `aura-orion-en` | English (US) Male voice – Deep, confident |
| `aura-arcas-en` | English (US) Male voice – Natural, conversational |
| `aura-perseus-en` | English (US) Male voice – Professional, engaging |

---

### Supported Audio Encodings & Containers

- **Encodings:** `mp3`, `linear16`, `opus`, `aac`, `flac`, `alaw`, `mulaw`
- **Containers:** `mp3`, `wav`, `ogg`, `none`

> **Note**
> 
> When using `linear16` PCM encoding for raw telephony or uncompressed audio, set `container` to `"wav"` to include standard WAV header bytes.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Workflow data used to supply `text` or `apiKey` dynamically via expressions |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted when speech synthesis succeeds. Contains audio binary buffer, mime type, and format details |
| `error` | `Error` | Emitted if API authentication, network request, or model generation fails |

---

### Example: MP3 Speech Generation

**Configuration**

```json
{
  "apiKey": "{{secrets.deepgramApiKey}}",
  "text": "Hello, welcome to our hotel. How can I help you today?",
  "model": "aura-asteria-en",
  "encoding": "mp3"
}
```

**Output**

```json
{
  "success": true,
  "contentType": "audio/mp3",
  "model": "aura-asteria-en",
  "encoding": "mp3",
  "data": "SUQzBAAAAAAAI1RTU0UAAAAPAAADTGF2ZjU4Lzc2LjEwMAAAAAAAAAAAAAAA...",
  "size": 42890
}
```

---

### Example: Linear16 WAV Speech Generation

**Configuration**

```json
{
  "apiKey": "{{secrets.deepgramApiKey}}",
  "text": "Hello, welcome to our hotel. How can I help you today?",
  "model": "aura-asteria-en",
  "encoding": "linear16",
  "container": "wav",
  "voice": "Welcome to our hotel. We hope you enjoy your stay."
}
```

**Output**

```json
{
  "success": true,
  "contentType": "audio/wav",
  "model": "aura-asteria-en",
  "encoding": "linear16",
  "container": "wav",
  "data": "UklGRiQAAABXQVZFZm10IBAAAAABAAEAQB8AAEAfAAABAAgAZGF0YQAAAAA...",
  "size": 154320
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Sample Workflow: Convert Text to MP3 Audio File

```json
{
  "nodes": [
    {
      "id": "manual-trigger",
      "type": "manual-trigger"
    },
    {
      "id": "generate-speech",
      "type": "deepgram",
      "config": {
        "apiKey": "{{secrets.deepgramApiKey}}",
        "text": "Hello, welcome to our hotel. How can I help you today?",
        "model": "aura-asteria-en",
        "encoding": "mp3"
      }
    },
    {
      "id": "log-output",
      "type": "log"
    }
  ]
}
```

### Common Patterns

- Customer Inquiry → AI Agent Text Response → Deepgram TTS → Audio Stream
- Article Published → Deepgram TTS (Aura Model) → Save MP3 to AWS S3 / Cloud Storage
- IVR Webhook → Deepgram TTS (Linear16/WAV) → Twilio / Telecom Audio Stream

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### API Key is required

**Cause**

The `apiKey` parameter was left blank or could not be resolved from workflow secrets.

**Solution**

Provide a valid Deepgram API key or reference it using expressions (e.g. `{{secrets.deepgramApiKey}}`).

---

### Text is required

**Cause**

The `text` parameter is empty or null.

**Solution**

Supply a non-empty string to convert to speech.

---

### Invalid Model Name

**Cause**

The specified `model` name does not exist or is not supported by your Deepgram subscription.

**Solution**

Use standard Deepgram Aura models such as `aura-asteria-en`, `aura-luna-en`, or `aura-orion-en`.

---

### Unauthorized / Invalid API Key (401)

**Cause**

The API key provided is invalid, revoked, or lacks project permissions for TTS operations.

**Solution**

Verify your key in the Deepgram Console dashboard and ensure TTS permissions are enabled.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Function](./function.md) – Process or format text before speech generation
- [Webhook Trigger](./webhook-trigger.md) – Receive incoming requests for voice synthesis
- [HTTP Request](./http-request.md) – Forward generated audio binary payloads to external APIs

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-11 | Initial release |

<!-- /SECTION: changelog -->
