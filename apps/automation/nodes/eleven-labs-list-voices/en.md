---
node_id: "eleven-labs-list-voices"
title: "Eleven Labs: List Voices"
description: "List the voices available to an ElevenLabs account through the ElevenLabs API."
category: "Generative AI & LLMs"
subcategory: "LLM Providers"
version: "1.0.0"
language: "en"
last_updated: "2026-09-01"
author: "Fusion Team"
tags:
  - elevenlabs
  - voices
  - text-to-speech
  - audio
  - generative-ai
  - voice-library
related_nodes:
  - eleven-labs-text-to-speech
  - eleven-labs-speech-to-speech
  - http-request
  - log
---

<!-- SECTION: header -->
# Eleven Labs: List Voices

> **Category:** Generative AI & LLMs | **Subcategory:** LLM Providers | **Type:** Action Node

List the voices available to an ElevenLabs account for use in speech generation and other voice workflows.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Eleven Labs: List Voices** node retrieves the available voices from the ElevenLabs voice library. The response can be used to discover voice IDs and metadata before sending a voice to text-to-speech or voice-conversion workflows.

### Key Features

- **Voice Discovery:** Retrieve voices available to the configured account
- **Voice Metadata:** Return voice IDs, names, categories, labels, descriptions, and previews when provided
- **Account-Aware Results:** List voices available to the authenticated ElevenLabs workspace
- **Secure Authentication:** Use an ElevenLabs API key through Fusion’s secret system
- **Downstream Integration:** Pass voice data to text-to-speech, filtering, database, or logging nodes
- **Error Routing:** Route authentication, network, and API failures to the error output

### Use Cases

- Find a voice ID before generating speech
- Build a voice-selection interface or workflow
- Filter voices by name, category, language, or labels
- Audit the voices available to an ElevenLabs account
- Select a voice dynamically for downstream audio generation

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `apiKey` | `string` | Yes | — | ElevenLabs API key used to authenticate the request |

### API Request

The node calls the ElevenLabs voices endpoint:

```text
GET https://api.elevenlabs.io/v1/voices
```

The API key is sent through the `xi-api-key` request header.

### API Key Authentication

Store the API key in Fusion’s secret system and reference it dynamically:

```json
{
  "apiKey": "{{secrets.elevenLabsApiKey}}"
}
```

Never commit a real ElevenLabs API key in a workflow example. The existing example workflow contains a literal key-shaped value and should be treated as exposed; revoke or rotate it and replace it with a secret reference.

### Response Options

The ElevenLabs API can return available voices with fields such as voice ID, name, category, labels, description, preview URL, and model information. The exact fields depend on the API version and account configuration.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Input

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` | Optional dynamic input containing an `apiKey` override |

### Success Output

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | ElevenLabs response containing the available voice records |

### Success Output Example

```json
{
  "voices": [
    {
      "voice_id": "voice-id-example",
      "name": "Example Voice",
      "category": "professional",
      "description": "A clear and expressive voice.",
      "labels": {
        "accent": "American",
        "gender": "female",
        "age": "middle-aged"
      },
      "preview_url": "https://example.com/voice-preview.mp3"
    }
  ]
}
```

### Error Output

Authentication failures, invalid requests, rate-limit responses, network errors, and ElevenLabs API failures are routed to the error output.

```json
{
  "success": false,
  "error": "ElevenLabs API request failed"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### List Available Voices

Use a secret-backed API key to retrieve the account’s available voices.

```json
{
  "apiKey": "{{secrets.elevenLabsApiKey}}"
}
```

### Select a Voice Downstream

Pass the successful response to a function or filter node, select a `voice_id`, and use that ID in an ElevenLabs text-to-speech workflow.

```text
Eleven Labs: List Voices → Filter or Function → Eleven Labs: Text to Speech
```

### Inspect the Response

Connect the node to a Log node to inspect available voice IDs, names, labels, and preview URLs.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: List ElevenLabs voices and inspect the response
```

### Common Patterns

- **Voice Discovery:** Manual Trigger → List Voices → Log
- **Voice Selection:** List Voices → Function → Text to Speech
- **Voice Filtering:** List Voices → Filter → Database
- **Audio Generation:** List Voices → Select Voice → Text to Speech → File or Media Output

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### API key is missing

**Cause:** The `apiKey` parameter is empty or the referenced secret is not configured.

**Solution:** Add a valid ElevenLabs API key to Fusion’s secret system and reference it with `{{secrets.elevenLabsApiKey}}`.

#### Unauthorized request

**Cause:** The key is invalid, expired, revoked, or sent in the wrong header.

**Solution:** Verify the key and confirm that the request uses the `xi-api-key` header through the node’s `apiKey` parameter.

#### No voices returned

**Cause:** The account has no accessible voices, or the API response is being filtered or transformed downstream.

**Solution:** Inspect the raw success output and verify the authenticated account and API plan.

#### Rate limit exceeded

**Cause:** The workflow exceeded the request limit for the ElevenLabs account or API plan.

**Solution:** Reduce request frequency, add a delay, or review the account limits.

#### Preview URL cannot be opened

**Cause:** A preview URL may be temporary, unavailable, or restricted by the account or network.

**Solution:** Use the returned voice metadata and request a new preview when supported by the ElevenLabs API.

### Error Codes

| Error | Cause | Solution |
|-------|-------|----------|
| `HTTP 401` | Missing or invalid API key | Configure a valid secret-backed API key |
| `HTTP 403` | Account or plan restriction | Check ElevenLabs account permissions and plan access |
| `HTTP 429` | Rate limit exceeded | Reduce request frequency and retry later |
| `HTTP 4xx` | Invalid request or account configuration | Verify the API key and endpoint |
| `HTTP 5xx` | ElevenLabs service failure | Retry after a short delay |
| `Network error` | Connection failure | Check connectivity and API availability |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **Eleven Labs: Text to Speech** - Generate speech using a selected voice
- **Eleven Labs: Speech to Speech** - Transform audio using an ElevenLabs voice
- [HTTP Request](../http-request/en.md) - Send generic HTTP requests
- **Log** - Inspect the list of returned voices

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-01 | Initial documentation |

<!-- /SECTION: changelog -->
