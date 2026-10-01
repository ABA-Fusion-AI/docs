---
node_id: "stomp-subscribe"
title: "STOMP Subscribe"
description: "Subscribe to STOMP destinations over WebSockets and trigger workflows on incoming messages with automatic JSON decoding and credential authentication."
category: "Triggers & Ingress"
subcategory: "Messaging & Event Ingress"
version: "1.0.0"
language: "en"
last_updated: "2026-10-01"
author: "Fusion Team"
tags:
  - stomp
  - subscribe
  - websocket
  - messaging
  - trigger
  - pubsub
  - real-time
  - event-driven
  - rabbitmq
  - activemq
related_nodes:
  - stomp-publish
  - nats-subscribe
  - websocket-trigger
  - function
  - log
---

<!-- SECTION: overview -->
# STOMP Subscribe

> **Category:** Triggers & Ingress&nbsp;&nbsp;|&nbsp;&nbsp;**Subgroup:** Messaging & Event Ingress&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Trigger Node

The **STOMP Subscribe** node connects your Fusion workflows to any [STOMP](https://stomp.github.io/) (Simple Text Oriented Messaging Protocol) message broker over WebSockets. As a **Trigger Node**, it listens continuously on a designated destination (topic, queue, or exchange) and fires a new workflow execution whenever an incoming message is received.

STOMP is widely used in enterprise and real-time web applications to decouple services and broadcast events. This node supports brokers such as **RabbitMQ** (via Web STOMP), **Apache ActiveMQ / Artemis**, and **Spring Boot WebSocket STOMP** messaging backends.

### Key Features

- **WebSocket-Based Connectivity:** Connects seamlessly to brokers using standard `ws://` (unencrypted) or `wss://` (TLS-encrypted) WebSocket endpoints.
- **Persistent Real-Time Streaming:** Maintains an active, persistent connection to the broker with automatic background reconnect handling (`reconnectDelay`).
- **Flexible Destination Routing:** Listen on standard pub/sub topics (e.g. `/topic/chat.room-505`), point-to-point queues (`/queue/orders`), or broker-specific exchange routes.
- **Smart Body Parsing:** Incoming message bodies are automatically parsed into structured JSON objects in `body`. If the payload is plain text or not JSON, it is passed safely as a string, while `rawBody` retains the exact original string.
- **STOMP Frame Headers:** Captures all protocol and user-defined headers (such as `destination`, `message-id`, `subscription`, `content-type`, and custom routing tags).
- **Authentication Support:** Securely authenticate against STOMP brokers using `login` and `passcode` credentials passed in the `CONNECT` frame.
- **Workflow Lifecycle Integration:** Supports pausing (temporarily unsubscribing while preserving configuration) and resuming connections smoothly.

### Use Cases

- **Real-Time Chat & Notification Ingestion:** Receive live chat messages, alerts, or mentions from web and mobile applications communicating over STOMP WebSockets.
- **Microservice Event Ingress:** Ingest asynchronous events emitted by backend services via RabbitMQ Web STOMP or ActiveMQ into automated workflows.
- **Live Financial & Telemetry Feeds:** Ingest streaming market updates, sensor readings, or telemetry feeds broadcasted on dedicated topic channels.
- **Cross-Service Queue Processing:** Trigger downstream business workflows (such as order fulfillment or CRM updates) whenever messages land on a STOMP `/queue/*` destination.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

Configure your STOMP WebSocket broker endpoint, target topic or queue destination, and optional authentication credentials in the node parameters panel.

### Parameters Reference

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `brokerURL` | `string` | ✅ Yes | — | WebSocket URL of the STOMP message broker. Must be a valid URL starting with `ws://` or `wss://` (e.g. `wss://broker.example.com/ws` or `ws://localhost:61614/stomp`). Supports expressions. |
| `topic` | `string` | ✅ Yes | — | The destination path to subscribe to (e.g. `/topic/chat.room-505` or `/queue/orders`). Must contain at least 1 character. |
| `login` | `string` | ❌ No | — | Username credential sent in the STOMP `CONNECT` frame for broker authentication. |
| `passcode` | `string` | ❌ No | — | Password or access token credential sent in the STOMP `CONNECT` frame. |
| `reconnectDelay` | `number` | ❌ No | `1000` | Wait time in milliseconds before attempting to reconnect when the broker connection drops. Defaults to `1000` ms (1 second). |

---

### Understanding STOMP Destinations

The `topic` parameter specifies the STOMP destination string. Different brokers use standard conventions for routing messages:

| Broker / Framework | Destination Pattern | Purpose | Example |
|--------------------|---------------------|---------|---------|
| **RabbitMQ (Web STOMP)** | `/topic/<name>` | Publish/subscribe topic exchange (`amq.topic`) | `/topic/chat.room-505` |
| | `/queue/<name>` | Direct point-to-point queue | `/queue/invoice-processing` |
| | `/exchange/<exchange>/<pattern>` | Custom AMQP exchange with custom routing keys | `/exchange/amq.direct/orders` |
| **Apache ActiveMQ / Artemis** | `/topic/<name>` | Broadcast channel to all active subscribers | `/topic/market-data` |
| | `/queue/<name>` | Load-balanced worker queue | `/queue/background-jobs` |
| **Spring Boot Messaging** | `/topic/<name>` | Multi-subscriber broadcast destinations | `/topic/announcements` |
| | `/queue/<name>` | User-specific or targeted destinations | `/queue/user-notifications` |

> [!TIP]
> **Leading Slashes:** Most STOMP brokers require destinations to begin with a forward slash (`/`). Ensure your destination starts with `/topic/`, `/queue/`, or the prefix specified by your broker configuration.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

As a **Trigger Node**, the **STOMP Subscribe** node has no input ports. It starts workflow runs automatically whenever a new message arrives from the broker.

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted whenever a message is received on the subscribed destination. |
| `error` | `object` | Emitted when connection fails, authentication is rejected, or the WebSocket connection drops unexpectedly. |

---

### Output Payload Examples

#### 1. Incoming Message Event (`success`)

When a message arrives on the subscribed destination, the node emits the following payload:

```json
{
  "topic": "/topic/chat.room-505",
  "body": {
    "sender": "sarah_k",
    "roomId": "room-505",
    "text": "Deployment completed successfully.",
    "priority": "normal"
  },
  "rawBody": "{\"sender\":\"sarah_k\",\"roomId\":\"room-505\",\"text\":\"Deployment completed successfully.\",\"priority\":\"normal\"}",
  "headers": {
    "destination": "/topic/chat.room-505",
    "content-type": "application/json",
    "subscription": "sub-0",
    "message-id": "msg-889104-prod"
  },
  "timestamp": "2026-10-01T09:30:15.240Z"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `topic` | `string` | The concrete destination string from the message frame (or fallback to configured `topic`). |
| `body` | `any` | The parsed message payload. If the body contains valid JSON, it is parsed into an object or array; otherwise, it contains the raw string. |
| `rawBody` | `string` | The original, unparsed string payload exactly as transmitted in the STOMP frame. |
| `headers` | `object` | Key-value mapping of all STOMP headers received with the message frame (e.g. `destination`, `message-id`, `content-type`, `subscription`). |
| `timestamp` | `string` | ISO 8601 formatted timestamp indicating when the message was received by Fusion. |

#### 2. Connection or Broker Error (`error`)

If the connection is interrupted or the broker rejects a frame, an error payload is emitted:

```json
{
  "error": "STOMP connection closed"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `error` | `string` | Descriptive error message explaining the connection failure, STOMP broker rejection, or WebSocket error. |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

The following example workflow demonstrates subscribing to a STOMP chat topic on a WebSocket broker and routing both successful messages and connection errors to a Log display node:

```fusion-workflow
src: example.workflow.json
title: Real-time STOMP message subscription and logging
```

### Complete Workflow Definition

```json
{
    "name": "STOMP Subscribe",
    "variables": {},
    "secrets": {},
    "nodes": [
        {
            "position": {
                "x": 26.377625861868694,
                "y": -82.28392939777132
            },
            "width": 72,
            "height": 72,
            "selected": false,
            "id": "kdkgafnyjfsgebo2bigf2lqp",
            "type": "trigger",
            "data": {
                "description": "Subscribe to STOMP destinations and trigger workflow on incoming messages. Supports WebSocket-based STOMP brokers with authentication.",
                "showRunningStatus": false,
                "_morphing": false,
                "name": "stomp-subscribe",
                "label": "STOMP Subscribe",
                "inputs": {},
                "outputs": {
                    "success": {
                        "label": "Success",
                        "isConnectable": null
                    },
                    "error": {
                        "label": "Error",
                        "isConnectable": null
                    }
                },
                "parameters": {
                    "brokerURL": "wss://broker.example.com/ws",
                    "topic": "/topic/chat.room-505",
                    "login": "guest",
                    "passcode": "guest"
                },
                "defaultOutput": null
            },
            "inputs": null,
            "outputs": null
        },
        {
            "position": {
                "x": 358.64850393188965,
                "y": -307.47421629213636
            },
            "width": 256,
            "height": 100,
            "dragHandle": ".drag-handle__custom",
            "selected": false,
            "id": "gu8fbatbo14bl0ieihde8qgb",
            "type": "display",
            "data": {
                "description": "Logs the input data to the console.",
                "showRunningStatus": false,
                "_morphing": false,
                "name": "log",
                "label": "Log",
                "inputs": {
                    "input": {
                        "label": "Input"
                    }
                },
                "outputs": {
                    "success": {
                        "label": "Success",
                        "isConnectable": false
                    }
                },
                "parameters": {},
                "defaultOutput": null
            },
            "inputs": null,
            "outputs": null
        }
    ],
    "connections": [
        {
            "type": "directed",
            "id": "xy-edge__kdkgafnyjfsgebo2bigf2lqpsuccess-gu8fbatbo14bl0ieihde8qgbinput",
            "data": {
                "isAnimated": false
            },
            "source": "kdkgafnyjfsgebo2bigf2lqp",
            "target": "gu8fbatbo14bl0ieihde8qgb",
            "sourceHandle": "success",
            "targetHandle": "input"
        },
        {
            "type": "directed",
            "id": "xy-edge__kdkgafnyjfsgebo2bigf2lqperror-gu8fbatbo14bl0ieihde8qgbinput",
            "data": {
                "isAnimated": false
            },
            "source": "kdkgafnyjfsgebo2bigf2lqp",
            "target": "gu8fbatbo14bl0ieihde8qgb",
            "sourceHandle": "error",
            "targetHandle": "input"
        }
    ],
    "status": "stopped",
    "tracingEnabled": true,
    "userId": "593a70dc-76b4-4abf-ac4f-de40c6d89861",
    "tenantId": "c4e72c92-6d3f-4ac3-b029-b722160d1088",
    "workspaceId": "9aae95bf-7bb8-4ec9-b268-e9337a62381f",
    "folderId": null,
    "version": 1,
    "createdAt": 1790789448405,
    "updatedAt": 1790846563052
}
```

---

### Practical Architecture Patterns

#### Pattern 1: Real-Time Chat & Incident Escalation

1. **STOMP Subscribe:** Listens to `/topic/support.alerts`.
2. **Function Node:** Inspects `{{input.body.priority}}` and extracts customer details.
3. **Condition / Filter Node:** Checks if `priority === "CRITICAL"`.
4. **Slack / PagerDuty Node:** Dispatches an emergency alert to the on-call engineering channel.

#### Pattern 2: RabbitMQ Web STOMP Order Pipeline

1. **STOMP Subscribe:** Listens to `/queue/ecommerce.orders` on `wss://rabbitmq.internal.company.com/ws`.
2. **Function Node:** Validates customer IDs and order item quantities from `{{input.body}}`.
3. **Database Action Node:** Inserts the transaction record into a PostgreSQL or MySQL ledger.
4. **STOMP Publish Node:** Publishes an order confirmation acknowledgement back to `/topic/orders.confirmed`.

#### Pattern 3: IoT Sensor Telemetry Monitor

1. **STOMP Subscribe:** Subscribes to `/topic/sensors.temperature`.
2. **Function Node:** Evaluates incoming temperature readings against threshold limits.
3. **Log / Alert Node:** Logs anomalies whenever readings exceed predefined safety boundaries.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Invalid URL: brokerURL must be a valid URL`

- **Cause:** The `brokerURL` parameter does not start with a valid protocol scheme or is formatted incorrectly.
- **Solution:** Ensure the URL starts with `ws://` (for plain WebSocket) or `wss://` (for secure WebSocket) and includes the full path (e.g. `wss://broker.example.com/ws` or `ws://127.0.0.1:15674/ws`). Do not use `http://` or `tcp://`.

#### `STOMP connection closed` or connection hangs indefinitely

- **Cause:** The WebSocket server is unreachable, the port is blocked by a firewall, or the broker does not have STOMP over WebSocket enabled.
- **Solution:**
  1. Confirm your message broker has its WebSocket plugin enabled (e.g. `rabbitmq-plugins enable rabbitmq_web_stomp` for RabbitMQ).
  2. Verify network connectivity from the Fusion host to the target host and port.
  3. Ensure intermediate proxies or load balancers (like Nginx, Cloudflare, or AWS ALB) are configured to support WebSocket upgrading (`Upgrade: websocket`).

#### `STOMP error: Bad CONNECT / Authentication failure`

- **Cause:** The broker requires authentication, but `login` or `passcode` was omitted, or invalid credentials were provided.
- **Solution:** Verify the username and password in your broker's user management interface and update the node's `login` and `passcode` fields.

#### Workflow never triggers even though messages are sent

- **Cause:** The destination path in `topic` does not match the exact destination used by the publisher.
- **Solution:**
  1. Verify destination spelling and leading slashes (e.g. `/topic/chat.room-505` vs `topic/chat.room-505`).
  2. Confirm whether your broker requires specific prefixes (e.g. RabbitMQ requires `/topic/` for topics and `/queue/` for queues).
  3. Inspect publisher logs to confirm messages are reaching the broker successfully.

#### `body` contains a string instead of an object

- **Cause:** The message payload published to the broker is not valid JSON (e.g. plain text string or XML).
- **Solution:** The node automatically provides `body` as a parsed object if JSON is valid; otherwise, it passes the raw text. If downstream nodes expect an object, ensure upstream publishers send valid JSON or use a Function node to format the content.

---

### Error Reference

| Error Indicator | Possible Cause | Recommended Resolution |
|-----------------|----------------|------------------------|
| `Invalid URL` | Protocol scheme is missing or not `ws://` / `wss://` | Verify `brokerURL` format. |
| `STOMP connection closed` | Remote broker disconnected, network timeout, or broker shutdown | Check broker health and network connection. |
| `STOMP error: Access Refused` | Missing or incorrect `login` / `passcode` credentials | Provide valid credentials in configuration. |
| `WebSocket connection failed` | Target port not listening or WebSocket upgrade rejected | Confirm broker Web STOMP plugin is active and port is accessible. |
| `Topic/destination is required` | The `topic` parameter is empty | Provide a valid destination name. |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [STOMP Publish](./stomp-publish.md) — Publish messages to STOMP destinations over WebSockets
- [NATS Subscribe](./nats-subscribe.md) — Subscribe to NATS messaging subjects
- [WebSocket Trigger](./web-socket-trigger.md) — Ingest generic raw WebSocket streaming events
- [Function](./function.md) — Transform and parse message payloads with custom JavaScript
- [Log](./log.md) — Inspect real-time message payloads and diagnostic logs

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-10-01 | Initial release of the STOMP Subscribe trigger node with WebSocket connectivity, automatic JSON decoding, credential authentication, and reconnection handling. |

<!-- /SECTION: changelog -->
