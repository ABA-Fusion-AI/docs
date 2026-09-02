---
node_id: "websocket-trigger"
title: "WebSocket Trigger"
description: "Triggers workflow when messages are received from a WebSocket server or client. Supports both client (connect to server) and server (host server) modes with automatic reconnection."
category: "triggers-ingress"
subcategory: "messaging-event-ingress"
version: "1.0.0"
language: "en"
last_updated: "2026-09-02"
author: "Fusion Team"
tags:
  - "websocket"
  - "trigger"
  - "messaging"
  - "events"
  - "realtime"
related_nodes:
  - "mqtt-subscriber"
  - "pusher-trigger"
  - "amqp-trigger"
  - "nats-subscribe"
---

<!-- SECTION: header -->

# WebSocket Trigger

> **Category:** Triggers & Ingress / Messaging & Event Ingress | **Type:** Trigger Node

Trigger workflows when WebSocket connection events or messages are received in client or server mode.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->

## Overview

The **WebSocket Trigger** node starts workflow execution from WebSocket events. It can operate as a client that connects to an external WebSocket server or as a server that accepts incoming WebSocket connections.

### Key Features

- Connect to external WebSocket servers in client mode
- Host a WebSocket endpoint in server mode
- Trigger workflows from WebSocket events and messages
- Support `ws://` and `wss://` connections
- Configure optional WebSocket protocols and headers
- Automatically reconnect when client connections are interrupted
- Configure reconnection interval and maximum reconnection attempts
- Handle connection open and close events
- Route WebSocket errors through the error output
- Configure binary data handling
- Support per-message compression

### Processing Flow

1. Configure the trigger in `client` or `server` mode.
2. In client mode, provide the WebSocket server URL.
3. In server mode, configure the WebSocket channel and server options.
4. The trigger starts listening for WebSocket activity.
5. Connection events or incoming messages are emitted through the workflow.
6. Errors are routed through the error output.
7. When enabled, interrupted client connections are automatically retried.

### Use Cases

- Receive real-time events from WebSocket APIs
- Trigger workflows from live application events
- Consume real-time messaging streams
- Listen for notifications from external services
- Build event-driven integrations
- Handle persistent WebSocket connections
- Integrate real-time systems with automation workflows

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|---|---|---|---|---|
| `type` | String | Yes | `client` | WebSocket operating mode. Use `client` to connect to an external server or `server` to host a WebSocket server. |
| `url` | String | No | — | WebSocket URL used in client mode, such as `ws://` or `wss://`. |
| `protocols` | Array | No | — | Optional WebSocket subprotocols to use when establishing the connection. |
| `headers` | Object | No | — | Optional headers sent when establishing a client connection. |
| `reconnect` | Boolean | No | `true` | Automatically attempt to reconnect when a client connection closes unexpectedly. |
| `reconnectInterval` | Number | No | `1000` | Delay in milliseconds between reconnection attempts. |
| `reconnectAttempts` | Number | No | — | Maximum number of reconnection attempts. |
| `binaryType` | String | No | `blob` | Binary data representation used by the WebSocket connection. |
| `perMessageDeflate` | Boolean | No | `true` | Enables WebSocket per-message compression when supported. |
| `channelName` | String | No | — | Channel name used when operating in server mode. |
| `responseType` | String | No | `none` | Configures the response behavior for server-mode messages. |
| `immediateResponse` | Any | No | — | Optional immediate response used by the server mode when configured. |
| `serverPort` | Number | No | — | Optional server port used when hosting a WebSocket server. |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->

## Inputs & Outputs

### Inputs

The **WebSocket Trigger** is a trigger node and does not require an incoming workflow connection.

WebSocket events are received directly from the configured client or server connection.

### Outputs

| Output | Type | Description |
|---|---|---|
| `success` | Object | Emits WebSocket connection events and received messages. |
| `error` | Error | Emits WebSocket connection, runtime, and reconnection errors. |

### Connection Events

When a client connection opens successfully, the success output can contain:

```json
{
  "type": "connection",
  "event": "open",
  "url": "wss://ws.postman-echo.com/raw",
  "timestamp": "2026-09-02T14:54:35.012Z"
}
```

When a connection closes, the success output can contain:

```json
{
  "type": "connection",
  "event": "close",
  "code": 1006,
  "reason": "",
  "url": "ws://127.0.0.1:59999",
  "timestamp": "2026-09-02T15:01:23.970Z"
}
```

### Error Output

Connection failures are routed through the error output.

For example:

```json
{
  "type": "error",
  "error": {
    "name": "Error",
    "message": "connect ECONNREFUSED 127.0.0.1:59999"
  },
  "url": "ws://127.0.0.1:59999"
}
```

When the configured reconnection limit is reached, the error output can contain:

```json
{
  "type": "error",
  "error": {
    "message": "Max reconnection attempts (2) reached",
    "type": "max_reconnect_attempts"
  },
  "url": "ws://127.0.0.1:59999"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->

## Examples

### Connect to a WebSocket Server

Configure the trigger as a WebSocket client:

```json
{
  "type": "client",
  "url": "wss://ws.postman-echo.com/raw",
  "reconnect": true,
  "reconnectInterval": 1000,
  "reconnectAttempts": 2,
  "binaryType": "blob",
  "perMessageDeflate": true
}
```

When the connection is established, the trigger emits a `connection` event with `event` set to `open`.

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: WebSocket Trigger Example
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->

## Troubleshooting

### WebSocket Connection Fails

**Cause:** The WebSocket server is unavailable, the URL is incorrect, or the connection is refused.

**Solution:** Verify that the WebSocket URL is valid and that the target server is reachable.

### Connection Closes With Code 1006

**Cause:** The WebSocket connection closed abnormally without a normal close frame.

**Solution:** Check the WebSocket server availability, network connectivity, and server-side logs when available.

### Maximum Reconnection Attempts Reached

**Cause:** The WebSocket connection repeatedly failed and reached the configured `reconnectAttempts` limit.

**Solution:** Verify the WebSocket server URL and availability. Increase `reconnectAttempts` if additional retry attempts are required.

### Reconnection Is Not Attempted

**Cause:** The `reconnect` option is disabled.

**Solution:** Enable `reconnect` and configure `reconnectInterval` and `reconnectAttempts` as required.

### Server Mode Produces No Output

**Cause:** Server mode waits for WebSocket client activity before producing workflow output.

**Solution:** Ensure a WebSocket client connects to the configured server endpoint and channel.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->

## Related Nodes

- **MQTT Subscriber** — Receive messages from MQTT topics.
- **Pusher Trigger** — Trigger workflows from Pusher events.
- **AMQP Trigger** — Receive events through AMQP.
- **NATS Subscribe** — Subscribe to NATS messaging subjects.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->

## Changelog

| Version | Date | Changes |
|---|---|---|
| `1.0.0` | `2026-09-02` | Initial release |

<!-- /SECTION: changelog -->