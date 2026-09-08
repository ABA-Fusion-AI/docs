---
node_id: "pusher-trigger"
title: "Pusher Trigger"
description: "Triggers workflow when events are received from Pusher channels."
category: "triggers-ingress"
subcategory: "messaging-event-ingress"
version: "1.0.0"
language: "en"
last_updated: "2026-09-08"
author: "Fusion Team"
tags:
  - "pusher"
  - "trigger"
  - "realtime"
  - "websockets"
  - "messaging"
  - "events"
related_nodes:
  - "pusher-action"
  - "web-socket-trigger"
  - "mqtt-subscriber"
  - "log"
---

<!-- SECTION: header -->

# Pusher Trigger

> **Category:** Triggers & Ingress / Messaging & Event Ingress | **Type:** Trigger Node

Triggers workflow executions in real time whenever subscribed events are received from Pusher channels.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->

## Overview

The **Pusher Trigger** node connects to the Pusher Channels real-time infrastructure via secure WebSockets (`forceTLS: true`). It subscribes to a specified channel and listens for incoming named events, instantly emitting the received data payload into your automation workflow.

### Key Features

- Real-time event listening using the Pusher Channels WebSocket protocol
- Enforced TLS encryption (`forceTLS: true`) for secure communication
- Precise event filtering by channel and event name
- Full connection lifecycle management (`setup`, `run`, `pause`, `resume`, and `stop`)
- Emits structured payloads containing event name, channel, data payload, and an ISO timestamp
- Dedicated error output port for handling subscription and connection errors

### Processing Flow

1. **Initialization:** The node initializes the Pusher client using the provided `appKey` and `cluster`.
2. **Connection & Subscription:** Upon workflow start (`run`), it connects to the Pusher gateway and subscribes to the configured `channel`.
3. **Event Binding:** Once the subscription succeeds, the node binds a listener to the configured `event`.
4. **Execution:** Incoming event payloads trigger downstream nodes via the `success` output.
5. **Error Handling:** Any subscription errors (such as unauthorized channel access) are routed through the `error` output.

### Use Cases

- **Live Notifications:** Trigger alert workflows when user actions or system events are published to Pusher.
- **Chat & Messaging:** Ingest chat messages or support room events in real time.
- **IoT & Telemetry:** Capture sensor updates and live device status broadcasts.
- **Order & Payment Updates:** React instantly to checkout completions and webhook-relayed payment events.
- **Cross-Service Sync:** Relay events from front-end applications or microservices into automated workflows.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|---|---|---|---|---|
| `appKey` | String | Yes | — | Pusher Application Key obtained from your Pusher Channels dashboard (**App Keys** tab). |
| `cluster` | String | Yes | — | Pusher cluster identifier hosting your application (e.g., `eu`, `us2`, `mt1`, `ap1`). |
| `channel` | String | Yes | — | Name of the channel to subscribe to (e.g., `test_fusion`, `notifications`, `chat-room-1`). |
| `event` | String | Yes | — | Name of the event to listen for on the subscribed channel (e.g., `user-registered`, `order-created`). |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->

## Inputs & Outputs

### Inputs

The **Pusher Trigger** is a trigger node and does not accept incoming workflow connections. It initiates workflows in response to external Pusher WebSocket events.

### Outputs

| Output | Type | Description |
|---|---|---|
| `success` | Object | Emits event data when a matching Pusher event is received on the subscribed channel. |
| `error` | Object | Emits subscription and connection failure details. |

### Output Schemas

#### Success Output (`success`)

When a subscribed event arrives, the node emits an object containing the event name, channel, parsed data payload, and an ISO timestamp:

```json
{
  "event": "user-registered",
  "channel": "test_fusion",
  "data": {
    "id": 1,
    "name": "Karim",
    "role": "admin"
  },
  "timestamp": "2026-09-08T09:45:56.106Z"
}
```

The `data` field preserves the original payload shape sent by Pusher (such as JSON objects, arrays, strings, or numbers).

#### Error Output (`error`)

If a subscription error occurs (e.g., attempting to subscribe to a private channel without client authorization), details are emitted via the error output:

```json
{
  "error": {
    "type": "subscription_error",
    "channel": "private-test_fusion",
    "data": {
      "type": "AuthError",
      "error": "Authorization failed"
    },
    "timestamp": "2026-09-08T09:50:00.000Z"
  }
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->

## Examples

### Basic Configuration

Configure the trigger to listen for user registration events:

```json
{
  "appKey": "43b22dd387a710ba648f",
  "cluster": "eu",
  "channel": "test_fusion",
  "event": "user-registered"
}
```

### Example Workflow

The following workflow listens for `user-registered` events on the `test_fusion` channel and logs the incoming payload directly to a **Log** node.

```fusion-workflow
src: example.workflow.json
title: Pusher Trigger with Log
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->

## Troubleshooting

### No Events Received When Triggering

**Cause:** 
- The event or channel name does not match the incoming event exactly (names are case-sensitive).
- The workflow has not been started or is currently paused.

**Solution:** 
- Verify in the **Pusher Debug Console** that the `channel` and `event` names match your node's parameters letter for letter.
- Ensure the workflow is in active/running status.

### Connection Fails / Invalid AppKey or Cluster

**Cause:** 
- The `appKey` does not exist or does not match the selected `cluster`.

**Solution:** 
- Go to **pusher.com** &rarr; select your App &rarr; click **App Keys**.
- Copy the exact `key` into `appKey` and the exact `cluster` (e.g., `eu`) into `cluster`.

### Subscription Error on `private-` or `presence-` Channels

**Cause:** 
- Channels prefixed with `private-` or `presence-` require an authentication endpoint (`/pusher/auth`) which is not configured for public client triggers.

**Solution:** 
- Use public channels (channels without `private-` or `presence-` prefixes) for standard trigger ingestion.
- Connect downstream handling to the node's **error** output port to catch and monitor subscription errors.

### Events Missed While Workflow Is Paused

**Cause:** 
- When the workflow is paused, the node unbinds event listeners to prevent unnecessary processing (`channel.unbind(event)`).

**Solution:** 
- Resuming the workflow automatically re-binds the event listener. Pusher does not queue historical events; only events published while the node is active are received.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->

## Related Nodes

- **Pusher Action** — Publish messages and events to Pusher channels from within a workflow.
- **WebSocket Trigger** — Receive raw messages from custom WebSocket servers or clients.
- **MQTT Subscriber** — Ingest messages from MQTT brokers and topics.
- **Log** — Inspect and log output payloads during development and debugging.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->

## Changelog

| Version | Date | Changes |
|---|---|---|
| `1.0.0` | `2026-09-08` | Initial release of the Pusher Trigger node. |

<!-- /SECTION: changelog -->
