---
node_id: "agora-token"
title: "Agora Token Gen"
description: "Generate an authentication token for Agora WebRTC."
category: "communication"
subcategory: "WebRTC"
version: "1.0.0"
language: "en"
last_updated: "2026-08-06"
author: "Fusion Team"
tags:
  - agora
  - webrtc
  - token
  - authentication
  - real-time
  - video
  - audio
related_nodes:
  - http-request
  - function
  - agorawebrtc
---

<!-- SECTION: overview -->
# Agora Token Gen

> **Category:** Communication &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Generate a short-lived RTC token for authenticating users in [Agora](https://www.agora.io) real-time video and audio channels. The token is computed server-side using your App ID, App Certificate, channel name, user ID, and role — and is required whenever token-based security is enabled in your Agora project.

### Use Cases

- **Secure Video Calls:** Generate a fresh token before each session and pass it to your frontend WebRTC client.
- **Dynamic Channel Access:** Issue role-specific tokens (`publisher` vs `subscriber`) based on user permissions.
- **Token Refresh Workflows:** Automatically regenerate tokens before expiry to maintain uninterrupted sessions.
- **Backend Auth Layer:** Use this node as a secure token endpoint so clients never need direct access to your App Certificate.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `appId` | `string` | Yes | — | Your Agora App ID. Found in the [Agora Console](https://console.agora.io) under your project settings. |
| `appCertificate` | `string` | Yes | — | Your Agora App Certificate. Required for token generation. Keep this secret — never expose it to the client. |
| `channelName` | `string` | Yes | — | The name of the Agora channel the user will join (e.g., `"test-channel"`). |
| `uid` | `number` | Yes | — | The unique user ID for this session (e.g., `12345`). Use `0` to let Agora assign a UID automatically. |
| `role` | `enum` | Yes | `subscriber` | The user's role in the channel. Either `publisher` (can send audio/video) or `subscriber` (can only receive). |
| `expiration` | `number` | No | `3600` | Token validity in **seconds**. Default is 3600 (1 hour). |

### Role Reference

| Role | Can Send Audio/Video | Can Receive Audio/Video |
|------|----------------------|-------------------------|
| `publisher` | Yes | Yes |
| `subscriber` | No | Yes |

> **Security tip:** Always store `appId` and `appCertificate` as workflow secrets, not hardcoded values.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` | Triggers the node. Parameters can be passed via expressions from upstream nodes. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted when the token is generated successfully. |
| `error` | `Error` | Emitted if any required parameter is missing or invalid. |

### Output Schema (`success`)

```json
{
  "token": "007eJxTYBBZeHTSr...",
  "uid": 12345,
  "channelName": "test-channel",
  "role": "subscriber",
  "expiresIn": 3600
}
```

| Field | Type | Description |
|-------|------|-------------|
| `token` | `string` | The generated Agora RTC token. Pass this to your WebRTC client to authenticate. |
| `uid` | `number` | The user ID the token was issued for. |
| `channelName` | `string` | The channel the token grants access to. |
| `role` | `string` | The role associated with this token (`publisher` or `subscriber`). |
| `expiresIn` | `number` | Token validity in seconds from the time of generation. |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Generate Agora RTC Token
```

### How it flows

1. **Manual Trigger:** Starts the workflow on demand.
2. **Agora Token Gen Node:** Receives the trigger, generates a token using the configured `appId`, `appCertificate`, `channelName` (`test-channel`), `uid` (`12345`), and `role` (`subscriber`).
3. **Log Node:** Displays the generated token and metadata.

### Common Patterns

- **On-Demand Token API:** Trigger via a Webhook node, pass `channelName` and `uid` as dynamic inputs, and return the token to the caller.
- **Token + WebRTC:** Chain with the Agora WebRTC node — generate a token first, then use it to join or start a session.
- **Role-Based Access:** Use an If/Else node to route to different token generations based on user permissions (publisher for hosts, subscriber for attendees).
- **Auto-Refresh:** Use a Cron or Interval node to regenerate tokens before they expire and push them to connected clients.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Invalid App ID or App Certificate`
- **Cause:** The `appId` or `appCertificate` is incorrect, copied partially, or belongs to a different Agora project.
- **Solution:** Copy both values directly from the [Agora Console](https://console.agora.io). Ensure they are from the same project.

#### Token rejected by Agora SDK on the client
- **Cause:** The token was generated for a different `channelName` or `uid` than what the client is using to join.
- **Solution:** Ensure the `channelName` and `uid` passed to this node exactly match the values used in the client SDK's `join()` call.

#### Token expires too quickly
- **Cause:** The `expiration` is set too low (e.g., `60` seconds).
- **Solution:** Increase `expiration` to a suitable value (e.g., `3600` for 1 hour, `86400` for 24 hours). Implement a token refresh workflow for long-running sessions.

#### `publisher` token not allowing the user to send audio/video
- **Cause:** The `role` parameter is set to `subscriber` instead of `publisher`.
- **Solution:** Set `role` to `publisher` for users who need to broadcast audio or video.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Agora WebRTC](./agorawebrtc.md) – Start or manage Agora WebRTC sessions
- [HTTP Request](./http-request.md) – Return the generated token to a frontend client via a webhook response
- [Function](./function.md) – Dynamically build channel names or UIDs before token generation

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-06 | Initial documentation |

<!-- /SECTION: changelog -->
