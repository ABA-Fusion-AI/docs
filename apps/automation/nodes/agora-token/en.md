---
node_id: "agora-token"
title: "Agora Token Gen"
description: "Generate dynamic RTC authentication tokens for Agora WebRTC audio and video streaming sessions."
category: "security"
subcategory: "identity-access"
version: "1.0.0"
language: "en"
last_updated: "2026-09-02"
author: "Fusion Team"
tags:
  - agora
  - webrtc
  - token
  - authentication
  - rtc
  - streaming
  - video
  - voice
related_nodes:
  - agora-event-trigger
  - jwt
  - http-request
  - webhook
---

<!-- SECTION: header -->
# Agora Token Gen

> **Category:** Security & Networking | **Type:** Action Node

Generate secure, time-limited **Agora RTC (Real-Time Communication) authentication tokens** for WebRTC audio calls, video conferences, interactive live streams, and real-time gaming rooms.
<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Agora Token Gen** node builds signed RTC authentication tokens using the official Agora token generation algorithm (`agora-access-token`). Whenever client applications (web, iOS, Android, Flutter, React Native, or Unity) connect to an Agora channel, Agora requires an authentication token to verify user identity, channel membership, permissions (publisher or subscriber), and session expiration.

This node securely signs token payloads server-side inside your workflow without exposing your **App Certificate** to frontend clients.

### Key Features

- **Standard Agora RTC Token Builder:** Implements standard RTC token construction compatible with Agora RTC SDK 4.x, 3.x, and WebRTC engines.
- **Publisher & Subscriber Roles:** Control privileges for hosts (can publish audio/video) vs. audience (view/listen only).
- **Flexible User Identification:** Supports integer `uid` (32-bit integer) or auto-assignment (`0`).
- **Configurable Expiration:** Set custom validity periods (in seconds) based on meeting or stream length.
- **Dynamic Expression Support:** Map channel names, user IDs, roles, and credentials dynamically from upstream triggers or database lookups using `{{input.field}}`.

### Common Use Cases

- **Video Conferencing & 1-on-1 Calls:** Generate publisher tokens for meeting participants when a room is created.
- **Interactive Live Streaming:** Issue host tokens (`publisher`) for streamers and audience tokens (`subscriber`) for viewers.
- **On-Demand Token API:** Build a token-dispensing webhook endpoint in Fusion that mobile or web apps call before joining a channel.
- **Automated Recording & Bots:** Create scheduled tokens for recording services or AI audio agents joining WebRTC channels.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

Configure the Agora credentials and session parameters required to build the token.

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `appId` | `string` | ✅ Yes | — | Your **Agora App ID** from the Agora Console. Identifies your Agora project. |
| `appCertificate` | `string` | ✅ Yes | — | Your **Agora App Certificate** (Primary certificate) from the Agora Console used to cryptographically sign the token. |
| `channelName` | `string` | ✅ Yes | `"room-general"` | The target WebRTC channel or room name to authorize (e.g. `room-101`, `conference-alpha`). |
| `uid` | `number` | ❌ No | `0` | Numerical User ID (32-bit unsigned integer). Setting `0` allows Agora to auto-assign a UID or permits any client-side UID. |
| `role` | `enum` | ❌ No | `"publisher"` | RTC privilege role:<br>• `publisher`: Host / Broadcaster (can publish audio and video streams).<br>• `subscriber`: Audience / Viewer (can subscribe to streams, cannot publish). |
| `expiration` | `number` | ❌ No | `3600` | Token lifetime duration in **seconds**. Default is `3600` (1 hour). Expiration timestamp is calculated as `currentTime + expiration`. |

---

### How to Get Your Agora Credentials

1. Sign in to the [Agora Console](https://console.agora.io/).
2. Navigate to **Project Management** in the left sidebar.
3. Select or create your project.
4. Copy the **App ID**.
5. Under project settings, locate **App Certificate** and enable/view the **Primary Certificate**.

> [!IMPORTANT]
> Keep your **App Certificate** secret. Never share it with frontend clients. Always generate tokens securely server-side using this node.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Incoming event data from upstream triggers (e.g., Webhook, HTTP trigger, Database query). Dynamic values can be used in configuration parameters using expression syntax (e.g., `{{input.channel}}`, `{{input.userId}}`). |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted when the token is successfully generated. |
| `error` | `Error` | Emitted if credential validation or token generation fails. |

### Output Payload (`success`)

When token generation succeeds, the node outputs the following structure:

```json
{
  "success": true,
  "token": "006c20ffa1d9121496094224ef5d0a5edc7IAC3O2B0mZ87j5q...",
  "channel": "room-general",
  "uid": 0,
  "expiresAt": 1788261477
}
```

#### Field Descriptions

- **`success`** (`boolean`): Indicates whether the generation was successful (`true`).
- **`token`** (`string`): The signed Agora RTC token string. Pass this token to your client SDK (`client.join(appId, channel, token, uid)`).
- **`channel`** (`string`): The channel name for which the token was issued.
- **`uid`** (`number`): The user ID embedded into the token.
- **`expiresAt`** (`number`): UNIX epoch timestamp (in seconds) when the token privileges expire.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Generate Host/Publisher Token

Generate a 2-hour publisher token for a specific room and user ID.

**Configuration:**
```json
{
  "appId": "c20ffa1d9121496094224ef5d0a5edc7",
  "appCertificate": "94bfde6442dd496ea91708c122ac3bc6",
  "channelName": "conference-room-42",
  "uid": 1001,
  "role": "publisher",
  "expiration": 7200
}
```

**Output:**
```json
{
  "success": true,
  "token": "006c20ffa1d9121496094224ef5d0a5edc7IAC...",
  "channel": "conference-room-42",
  "uid": 1001,
  "expiresAt": 1788268677
}
```

---

### Example 2: Dynamic Token Dispenser for Webhooks

Accept dynamic channel names and user roles from an incoming Webhook request.

**Incoming Webhook Body:**
```json
{
  "room": "livestream-99",
  "userId": 505,
  "userRole": "subscriber"
}
```

**Node Configuration:**
```json
{
  "appId": "c20ffa1d9121496094224ef5d0a5edc7",
  "appCertificate": "94bfde6442dd496ea91708c122ac3bc6",
  "channelName": "{{input.room}}",
  "uid": "{{input.userId}}",
  "role": "{{input.userRole}}",
  "expiration": 3600
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Generate Agora RTC Token on Demand
```

### Sample Workflow: Webhook Token Dispenser API

A mobile client sends an HTTP POST request to request an Agora RTC token before joining a video call. Fusion validates the request, generates the token using **Agora Token Gen**, and returns the token in the HTTP response.

```json
{
  "nodes": [
    {
      "id": "webhook-trigger",
      "type": "webhook",
      "position": { "x": 200, "y": 200 },
      "data": {
        "path": "/api/agora/token",
        "method": "POST"
      }
    },
    {
      "id": "agora-token",
      "type": "agora-token",
      "position": { "x": 450, "y": 200 },
      "data": {
        "parameters": {
          "appId": "{{secrets.AGORA_APP_ID}}",
          "appCertificate": "{{secrets.AGORA_APP_CERTIFICATE}}",
          "channelName": "{{input.body.channelName}}",
          "uid": "{{input.body.uid}}",
          "role": "{{input.body.role}}",
          "expiration": 3600
        }
      }
    },
    {
      "id": "respond-webhook",
      "type": "respond-to-webhook",
      "position": { "x": 700, "y": 200 },
      "data": {
        "statusCode": 200,
        "responseBody": {
          "token": "{{input.token}}",
          "channel": "{{input.channel}}",
          "uid": "{{input.uid}}",
          "expiresAt": "{{input.expiresAt}}"
        }
      }
    }
  ]
}
```

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues & Solutions

#### `CAN_NOT_GET_GATEWAY_SERVER (101)` or Invalid App ID
- **Cause:** The `appId` is invalid, contains spaces, or does not exist in the Agora Console.
- **Solution:** Verify the App ID in your Agora Console under Project Management.

#### `DYNAMIC_USE_STATIC_KEY (109)` or `TOKEN_EXPIRED (109)`
- **Cause:** The Agora project has **App Certificate** enabled, but the client attempted to join without a token, or the token has expired.
- **Solution:** Ensure the client passes the token generated by this node to `client.join()`, and increase `expiration` if sessions last longer than 1 hour.

#### `INVALID_TOKEN (110)`
- **Cause:** The token was generated with a different `channelName` or `uid` than what the client provided when calling `join()`.
- **Solution:** Ensure `channelName` and `uid` match exactly between this node and the client SDK's `join(appId, channelName, token, uid)` call. If `uid: 0` is used, the client can use any integer UID.

#### Client cannot publish audio or video stream
- **Cause:** The token was generated with `role: "subscriber"`.
- **Solution:** Set `role` to `"publisher"` if the user needs host/broadcasting permissions.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Agora Event Trigger](./agora-event-trigger.md) – Trigger workflows on Agora Webhook events (e.g. channel destroy, user leave).
- [Webhook](../triggers/webhook.md) – Receive incoming token requests from client apps.
- [Respond to Webhook](../utilities/respond-to-webhook.md) – Send HTTP responses containing generated tokens back to callers.
- [JWT](./jwt.md) – Generate JSON Web Tokens for general API authentication.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-02 | Initial release with RTC token generation with UID, publisher/subscriber roles, and custom expiration. |

<!-- /SECTION: changelog -->
