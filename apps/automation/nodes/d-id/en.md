---
node_id: "did"

title: "D-ID: Generate Avatar Video"

description: "Generate avatar videos from text or audio using the D-ID API."

category: "AI / Video Generation"

version: "1.0.0"

language: "en"

last_updated: "2026-09-09"

author: "Fusion Team"

tags:

- d-id

- avatar

- video-generation

- ai-video

- text-to-video

- audio-to-video

- talking-avatar

- api

related_nodes:

- http-request

- function

- if

---

**# D-ID: Generate Avatar Video**

> **\*\*Category:\*\*** ai-nodes | **\*\*Type:\*\*** Action Node

Generate AI avatar videos using the **\*\*D-ID\*\*** Talks API.

The **\*\*D-ID: Generate Avatar Video\*\*** node sends a `POST` request to the D-ID `/talks` endpoint using a source image URL and either text-based or audio-based script input.

For text scripts, the node can send plain text using `input` and optionally include `ssml`. For audio scripts, it sends an external audio file URL using `audio_url`.

**### Supported Features**

\- Generate talking-avatar videos using D-ID

\- Use text as the avatar speech source

\- Use an external audio URL as the avatar speech source

\- Support optional SSML for text scripts

\- Use a remote source image through `sourceUrl`

\- Configure MP4 video output

\- Enable or disable fluent mode

\- Configure audio padding

\- Enable or disable stitching

\- Configure custom driver expressions

\- Authenticate using D-ID Basic Authentication

\- Return the parsed D-ID API response directly

**### Use Cases**

\- Generate AI presenter videos from text

\- Animate a portrait or avatar image

\- Create talking avatars from prerecorded audio

\- Generate automated presentation videos

\- Create personalized video messages

\- Integrate avatar-video generation into automated workflows

**---**

**## Configuration**

**### Base Parameters**

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `apiKey` | `string` | ✅ Yes | — | D-ID API key. Must contain at least one character. |
| `script` | `object` | ✅ Yes | — | Defines whether the avatar uses text or audio. |
| `sourceUrl` | `string` | ✅ Yes | — | URL of the source image. Must contain at least one character. |
| `config` | `object` | ❌ No | — | Optional video-generation configuration. |

**### Script Parameters**

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `script.type` | `enum` | ✅ Yes | — | `text` or `audio`. |
| `script.input` | `string` | ❌ No | — | Text included when `script.type` is `text`. |
| `script.ssml` | `string` | ❌ No | — | Optional SSML included for text scripts. |
| `script.audioUrl` | `string` | ❌ No | — | External audio URL included when `script.type` is `audio`. |

**### Configuration Parameters**

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `config.resultFormat` | `enum` | ❌ No | — | Video result format. Documented here as `mp4`. |
| `config.fluent` | `boolean` | ❌ No | — | Optional fluent setting. |
| `config.padAudio` | `number` | ❌ No | — | Optional audio padding. |
| `config.stitch` | `boolean` | ❌ No | — | Optional stitching setting. |
| `config.driverExpressions` | `record` | ❌ No | — | Optional driver expressions. |

**---**

**## Operations**

| Operation | Endpoint | Method | Description |
| --------- | -------- | ------ | ----------- |
| Generate Avatar Video | `/talks` | `POST` | Generate a D-ID talking-avatar video from text or audio. |

The fixed endpoint is:

```text
https://api.d-id.com/talks
```

**---**

**## Request Body Construction**

**### Text Script**

```json
{
  "script": {
    "type": "text",
    "input": "Hello, welcome to our presentation."
  },
  "source_url": "https://example.com/avatar.jpg"
}
```

When `ssml` is supplied for a text script, it is also added to the `script` object.

**### Audio Script**

```json
{
  "script": {
    "type": "audio",
    "audio_url": "https://example.com/audio.mp3"
  },
  "source_url": "https://example.com/avatar.jpg"
}
```

The node transforms:

```text
audioUrl → audio_url
sourceUrl → source_url
```

**### Video Configuration**

```json
{
  "config": {
    "result_format": "mp4",
    "fluent": true,
    "pad_audio": 0,
    "stitch": true
  }
}
```

Configuration names are transformed as follows:

```text
resultFormat → result_format
padAudio → pad_audio
driverExpressions → driver_expressions
```

Boolean `fluent` and `stitch` values are included whenever they are not `undefined`. `padAudio` is also included whenever defined, including `0`.

**---**

**## Inputs & Outputs**

**### Inputs**

The node does not use incoming workflow data. All parameters come from node configuration.

**### Outputs**

The node returns the parsed D-ID JSON response directly.

It does not wrap or normalize the returned response.

**### Output Example**

The exact response depends on the D-ID API. The node returns that JSON object unchanged.

**---**

**## Configuration Examples**

**### Generate Video From Text**

```json
{
  "apiKey": "YOUR_DID_API_KEY",
  "script": {
    "type": "text",
    "input": "Hello, welcome to our presentation."
  },
  "sourceUrl": "https://example.com/avatar.jpg"
}
```

**### Generate Video From Text With SSML**

```json
{
  "apiKey": "YOUR_DID_API_KEY",
  "script": {
    "type": "text",
    "input": "Hello, welcome to our presentation.",
    "ssml": "<speak>Hello, welcome to our presentation.</speak>"
  },
  "sourceUrl": "https://example.com/avatar.jpg"
}
```

**### Generate Video From Audio**

```json
{
  "apiKey": "YOUR_DID_API_KEY",
  "script": {
    "type": "audio",
    "audioUrl": "https://example.com/audio.mp3"
  },
  "sourceUrl": "https://example.com/avatar.jpg"
}
```

**### Generate MP4 Video**

```json
{
  "apiKey": "YOUR_DID_API_KEY",
  "script": {
    "type": "text",
    "input": "This video was generated automatically."
  },
  "sourceUrl": "https://example.com/avatar.jpg",
  "config": {
    "resultFormat": "mp4",
    "fluent": true,
    "padAudio": 0,
    "stitch": true
  }
}
```

**---**

**## Workflow Integration**

**### Common Patterns**

\- Text Generation → D-ID — create an avatar video request from generated text

\- Text-to-Speech → Audio URL → D-ID — animate an avatar using generated audio

\- Database → Function → D-ID — generate personalized avatar videos

\- Trigger → D-ID → Notification — submit video generation and pass the returned data downstream

\- D-ID → Function — process the returned D-ID talk information

**---**

**## Error Handling**

**### D-ID API Error**

For non-success HTTP responses, the node throws:

```text
D-ID API error (Status: <status>): <message>
```

The error message is selected in this order:

```text
errorData.error.message
errorData.message
response.statusText
```

If the failed response cannot be parsed as JSON, the node falls back to `response.statusText`.

**---**

**## Troubleshooting**

**### "D-ID API error (Status: 401): ..."**

**\*\*Cause\*\***

The D-ID API rejected the supplied credentials.

**\*\*Solution\*\***

Verify `apiKey`.

---

**### Text Script Does Not Contain Input**

**\*\*Cause\*\***

`script.type` is `text`, but `script.input` is empty or undefined.

**\*\*Solution\*\***

Provide a non-empty `input` value.

---

**### Audio Script Does Not Contain an Audio URL**

**\*\*Cause\*\***

`script.type` is `audio`, but `script.audioUrl` is empty or undefined.

**\*\*Solution\*\***

Provide a valid external audio URL.

---

**### Source Image Request Fails**

Verify that `sourceUrl` points to a resource accessible to D-ID.

**---**

**## Security**

Authentication uses HTTP Basic Authentication:

```text
Authorization: Basic <base64(apiKey:)>
```

The credential is constructed from:

```text
Buffer.from(`${apiKey}:`).toString("base64")
```

The request also includes:

```text
Content-Type: application/json
```

For production workflows:

\- Store the API key using protected secrets or expressions

\- Avoid exposing Authorization headers in logs

\- Use trusted HTTPS source and audio URLs

\- Review externally supplied URLs before passing them to D-ID

**---**

**## Notes**

The node uses:

```text
POST https://api.d-id.com/talks
```

Supported script types are:

```text
text
audio
```

For text scripts, `input` and `ssml` are included only when truthy.

For audio scripts, `audio_url` is included only when `audioUrl` is truthy.

The node transforms:

```text
sourceUrl → source_url
audioUrl → audio_url
resultFormat → result_format
padAudio → pad_audio
driverExpressions → driver_expressions
```

The node returns the D-ID JSON response directly.

The node does not:

\- Poll the generated talk until processing completes

\- Download the generated video

\- Upload source images

\- Upload audio files

\- Generate text automatically

\- Generate audio automatically

\- Retry failed requests

\- Cache responses

The `stop()` method performs no cleanup logic.

This documentation intentionally excludes WebM usage as requested.

**---**

**## Changelog**

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-09-09 | Initial release |
