---
node_id: "websocket-send"
title: "WebSocket Send"
description: "Sends messages to a WebSocket server. Supports text and binary messages, connection reuse, and response waiting."
category: "communication-messaging"
subcategory: "chat-collaboration"
version: "1.0.0"
language: "en"
last_updated: "2026-09-03"
author: "Fusion Team"
tags:
  - "websocket"
  - "messaging"
  - "realtime"
  - "communication"
  - "send"
related_nodes:
  - "websocket-trigger"
  - "pusher-action"
  - "xmpp"
---

<!-- SECTION: header -->

# WebSocket Send

> **Category:** Communication & Messaging / Chat & Collaboration | **Type:** Action Node

Send messages to WebSocket servers with support for text and binary data, response waiting, custom connection options, and connection reuse.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->

## Overview

The **WebSocket Send** node connects to a WebSocket server and sends messages as part of a workflow. Messages can be provided directly through the node configuration or generated from incoming workflow data.

The node can optionally wait for a response, reuse an existing WebSocket connection, configure custom protocols and headers, handle binary data, and control connection and response timeouts.

### Key Features

- Connect to `ws://` and `wss://` WebSocket servers
- Send configured text messages
- Use incoming workflow data when no message is configured
- Serialize incoming objects as JSON before sending
- Wait for and return WebSocket responses
- Parse JSON responses automatically when possible
- Support text and binary messages
- Configure WebSocket subprotocols
- Send custom connection headers
- Configure binary data handling
- Enable or disable per-message compression
- Reuse active WebSocket connections
- Close connections after sending when required
- Configure connection and response timeouts
- Return structured WebSocket error information

### Processing Flow

1. The node receives data from the previous workflow node.
2. It prepares the message using the configured `message` value when provided.
3. If `message` is empty, the incoming workflow data is used instead.
4. Non-string input data is serialized to JSON when possible.
5. The node opens a new WebSocket connection or reuses an existing connection when enabled.
6. The prepared message is sent to the WebSocket server.
7. If `waitForResponse` is enabled, the node waits for one response from the server.
8. Text responses are parsed as JSON when possible.
9. The node returns the send result or structured WebSocket error information.

### Use Cases

- Send real-time messages to WebSocket APIs
- Integrate workflow data with WebSocket services
- Send application events to real-time systems
- Exchange request-response messages over WebSockets
- Push JSON payloads to WebSocket servers
- Maintain reusable WebSocket connections
- Connect automation workflows to event-driven applications

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|---|---|---|---|---|
| `url` | String | Yes | — | WebSocket server URL. Supports WebSocket endpoints such as `ws://` and `wss://`. |
| `message` | String | No | — | Message to send. When empty, the node uses the incoming workflow data. |
| `protocols` | Array | No | — | Optional WebSocket subprotocols used when establishing the connection. |
| `headers` | Object | No | — | Optional HTTP headers sent during the WebSocket connection handshake. |
| `binaryType` | String | No | — | Binary data representation. Supported values are `blob` and `arraybuffer`. |
| `perMessageDeflate` | Boolean | No | `true` | Enables WebSocket per-message compression. |
| `closeAfterSend` | Boolean | No | `false` | Closes the WebSocket connection after the message is sent. |
| `reuseConnection` | Boolean | No | `false` | Reuses an existing open WebSocket connection when available. |
| `waitForResponse` | Boolean | No | `false` | Waits for a WebSocket response after sending the message. |
| `responseTimeout` | Number | No | `30000` | Maximum time in milliseconds to wait for a response. Minimum value is `1000`. |
| `timeout` | Number | No | `30000` | Maximum time in milliseconds allowed to establish the WebSocket connection. Minimum value is `1000`. |

### Message Selection

When `message` contains a value, that value is sent directly.

When `message` is empty, the node uses the incoming workflow data:

- Strings are sent directly.
- Buffers are sent as binary data.
- ArrayBuffers are converted to binary data.
- Other values are serialized using JSON when possible.
- If JSON serialization fails, the value is converted to a string.

### Response Handling

When `waitForResponse` is disabled, the node returns immediately after the message is sent.

When `waitForResponse` is enabled, the node waits for one WebSocket message:

- JSON text responses are parsed into objects.
- Other text responses remain strings.
- Binary responses are converted to Base64 when possible.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->

## Inputs & Outputs

### Inputs

The node accepts workflow data through its `input` connection.

If the `message` parameter is empty, this incoming data becomes the message sent to the WebSocket server.

For example, incoming data such as:

```json
{
  "tick": 1,
  "timestamp": 1788444550356
}
```

is serialized before being sent:

```json
"{\"tick\":1,\"timestamp\":1788444550356}"
```

### Outputs

| Output | Type | Description |
|---|---|---|
| `success` | Object | Returns information about the sent message and, when enabled, the received response. |
| `error` | Error | Error output available for workflow-level execution failures. |

### Successful Send

When `waitForResponse` is disabled, a successful execution returns information similar to:

```json
{
  "success": true,
  "url": "wss://ws.postman-echo.com/raw",
  "message": "Hello Fusion",
  "isBinary": false,
  "sentAt": "2026-09-03T14:07:23.481Z"
}
```

### Successful Send With Response

When `waitForResponse` is enabled:

```json
{
  "success": true,
  "url": "wss://ws.postman-echo.com/raw",
  "message": "Hello Fusion",
  "isBinary": false,
  "sentAt": "2026-09-03T14:08:00.000Z",
  "response": "Hello Fusion",
  "responseIsBinary": false,
  "receivedAt": "2026-09-03T14:08:00.100Z"
}
```

### WebSocket Error

WebSocket connection and send failures are returned as structured data:

```json
{
  "success": false,
  "error": {
    "type": "websocket_error",
    "message": "connect ECONNREFUSED 127.0.0.1:59999",
    "url": "ws://127.0.0.1:59999",
    "timestamp": "2026-09-03T14:10:00.000Z"
  }
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->

## Examples

### Send a Message and Wait for a Response

Configure the node with:

```json
{
  "url": "wss://ws.postman-echo.com/raw",
  "message": "Hello Fusion",
  "protocols": [],
  "headers": {},
  "binaryType": "blob",
  "perMessageDeflate": true,
  "closeAfterSend": false,
  "reuseConnection": false,
  "waitForResponse": true,
  "responseTimeout": 30000,
  "timeout": 30000
}
```

The message is sent to the WebSocket server and the node waits for one response before completing.

### Send Incoming Workflow Data

Leave `message` empty to use data received from the previous node.

For object input, the node serializes the value as JSON before sending it to the WebSocket server.

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: WebSocket Send Example
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->

## Troubleshooting

### WebSocket Connection Is Refused

**Cause:** The WebSocket server is unavailable, the port is closed, or the URL points to an unreachable endpoint.

**Solution:** Verify the WebSocket URL, server availability, port, and network connectivity.

A failed connection can return:

```text
connect ECONNREFUSED
```

### Response Timeout

**Cause:** `waitForResponse` is enabled but the server does not return a message before `responseTimeout` expires.

**Solution:** Verify that the server sends a response or increase `responseTimeout`.

### Connection Timeout

**Cause:** The WebSocket connection cannot be established before the configured `timeout`.

**Solution:** Verify server availability and network connectivity, or increase the timeout value.

### Timeout Validation Fails

**Cause:** `responseTimeout` or `timeout` is lower than the minimum accepted value.

**Solution:** Configure a value of at least `1000` milliseconds.

### No Configured Message

**Cause:** The `message` parameter is empty.

**Solution:** This is supported behavior. The node uses incoming workflow data as the message instead.

### JSON Response Is Returned as Text

**Cause:** The WebSocket server response is not valid JSON.

**Solution:** Valid JSON responses are parsed automatically. Other text responses are returned as strings.

### Connection Reuse Does Not Occur

**Cause:** `reuseConnection` is disabled or no reusable open connection exists.

**Solution:** Enable `reuseConnection` and ensure the connection remains open between sends.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->

## Related Nodes

- **WebSocket Trigger** — Trigger workflows from WebSocket connections and incoming messages.
- **Pusher Action** — Send data through Pusher-based integrations.
- **XMPP** — Integrate workflows with XMPP messaging services.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->

## Changelog

| Version | Date | Changes |
|---|---|---|
| `1.0.0` | `2026-09-03` | Initial release |

<!-- /SECTION: changelog -->