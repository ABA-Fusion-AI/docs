---
node_id: "stomp-publish"
title: "STOMP Publish"
description: "Publish messages to STOMP destinations over WebSockets with dynamic payload routing, automatic JSON serialization, and credential authentication."
category: "Communication & Messaging"
subcategory: "Queues & Event Streams"
version: "1.0.0"
language: "en"
last_updated: "2026-10-01"
author: "Fusion Team"
tags:
  - stomp
  - publish
  - websocket
  - messaging
  - pubsub
  - queues
  - event-driven
  - real-time
  - rabbitmq
  - activemq
related_nodes:
  - stomp-subscribe
  - nats-publish
  - function
  - manual-trigger
  - log
---

<!-- SECTION: overview -->
# STOMP Publish

> **Category:** Communication & Messaging&nbsp;&nbsp;|&nbsp;&nbsp;**Subgroup:** Queues & Event Streams&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

The **STOMP Publish** node produces and sends messages to destinations (topics, queues, or exchanges) on a [STOMP](https://stomp.github.io/) (Simple Text Oriented Messaging Protocol) message broker over WebSockets. It acts as an event emitter within event-driven architectures, enabling Fusion workflows to broadcast real-time updates, dispatch commands to microservices, and push notifications to connected web or mobile clients.

This node is compatible with any WebSocket-enabled STOMP broker, including **RabbitMQ** (via the Web STOMP plugin), **Apache ActiveMQ / Artemis**, and **Spring Boot WebSocket Messaging** applications.

### Key Features

- **WebSocket-Based Transmission:** Sends messages over unencrypted (`ws://`) or TLS-secured (`wss://`) WebSocket connections directly from the workflow runtime.
- **Dynamic Routing Overrides:** Configure static destination topics and messages in the parameters panel, or dynamically override both `topic` and `message` at runtime using incoming data or workflow expressions.
- **Automatic Payload Serialization:** Automatically serializes incoming JavaScript objects and arrays into clean JSON strings before transmitting them in the STOMP frame body.
- **Resilient Connection Lifecycle:** Manages the STOMP client connection, waits for the socket to establish an open state (with a 10-second safety timeout), and automatically handles disconnections with configurable retry delays (`reconnectDelay`).
- **Broker Authentication:** Supports `login` and `passcode` credentials passed in the STOMP `CONNECT` frame for secured brokers.
- **Immediate Delivery Dispatch:** Emits confirmation upon publishing, returning the destination, message body, and broker endpoint details.

### Use Cases

- **Real-Time Client Broadcasts:** Push live announcements, chat messages, or status updates to web users connected to a `/topic/chat.*` or `/topic/notifications` channel.
- **Microservice Task Dispatching:** Enqueue background jobs or transaction tasks to RabbitMQ queues (e.g. `/queue/order-processing`) for worker consumption.
- **Responding to Inbound Subscriptions:** Pair with [STOMP Subscribe](./stomp-subscribe.md) to ingest incoming inquiries, compute results, and publish response frames back to a dedicated response destination.
- **IoT & Remote Device Commands:** Broadcast control signals and setpoint configurations to connected hardware gateways listening on MQTT or STOMP exchanges.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

Configure your broker connection endpoint, default destination topic, and message content in the node parameters panel.

### Parameters Reference

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `brokerURL` | `string` | ✅ Yes | — | WebSocket URL of the STOMP message broker. Must be a valid URL starting with `ws://` or `wss://` (e.g. `wss://broker.example.com/ws` or `ws://localhost:61614/stomp`). Supports expressions. |
| `topic` | `string` | ✅ Yes | — | Default destination path to publish messages to (e.g. `/topic/chat.room-505` or `/queue/orders`). Can be dynamically overridden by incoming upstream data. |
| `message` | `string` | ❌ No | — | Message body to send. Can be a static string, a JSON template, or left empty if supplied dynamically from upstream nodes. |
| `login` | `string` | ❌ No | — | Username credential sent in the STOMP `CONNECT` frame for broker authentication. |
| `passcode` | `string` | ❌ No | — | Password or access token credential sent in the STOMP `CONNECT` frame. |
| `reconnectDelay` | `number` | ❌ No | `1000` | Wait time in milliseconds before attempting to reconnect if the connection to the broker is lost. Defaults to `1000` ms (1 second). |

---

### Understanding STOMP Destinations

The `topic` field specifies where the message is sent. Depending on your broker, standard naming conventions apply:

| Broker / Framework | Destination Pattern | Purpose | Example |
|--------------------|---------------------|---------|---------|
| **RabbitMQ (Web STOMP)** | `/topic/<name>` | Broadcast to all subscribers on a topic exchange (`amq.topic`) | `/topic/chat.room-505` |
| | `/queue/<name>` | Point-to-point delivery to queue workers | `/queue/order-processing` |
| | `/exchange/<exchange>/<routing_key>` | Publish to a custom AMQP exchange with a specific routing key | `/exchange/amq.direct/critical-alerts` |
| **Apache ActiveMQ / Artemis** | `/topic/<name>` | Publish/subscribe broadcast channel | `/topic/system-status` |
| | `/queue/<name>` | Load-balanced worker queue | `/queue/email-jobs` |
| **Spring Boot Messaging** | `/topic/<name>` | Broadcast to clients subscribed to `@SendTo` destinations | `/topic/dashboard-metrics` |
| | `/queue/<name>` | User-targeted destinations | `/queue/user-101-inbox` |

> [!TIP]
> **Leading Slashes:** Most STOMP brokers require destinations to begin with a forward slash (`/`). Ensure your destination starts with `/topic/`, `/queue/`, or your broker's expected prefix.

---

### Dynamic Payload Routing & Message Resolution

The **STOMP Publish** node evaluates destination and message content using a flexible resolution hierarchy:

#### 1. Destination (`topic`) Resolution Order

1. **Incoming Payload:** If the incoming data object from the upstream node contains a `topic` property (e.g. `{{input.topic}}`), that value is used as the destination.
2. **Node Configuration:** If no `topic` is present in the payload, the node falls back to the static `topic` value configured in the parameters panel.
3. If neither is available, the node halts with an error: `Topic/destination is required`.

#### 2. Message (`message`) Resolution Order

1. **Incoming Payload Property:** If the incoming data object has a `message` property (e.g. `{{input.message}}`), that string is published.
2. **Node Configuration:** If no `message` property exists on the input, the node uses the static `message` field configured in the parameters panel.
3. **String Payload Fallback:** If the upstream node outputs a raw string, that string is used directly as the message body.
4. **Automatic Object Serialization:** If the upstream node outputs an object or array without a dedicated `message` property, the node automatically serializes the entire input object to a JSON string using `JSON.stringify(data)`.
5. If no message can be resolved, the node returns an error: `Message is required`.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Incoming data from the preceding node. Can be an object containing `{ "topic": "...", "message": "..." }`, an object with custom data to serialize automatically as JSON, or a raw string. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted when the message is accepted and published to the STOMP broker. |
| `error` | `object` | Emitted when connection times out (exceeding 10 seconds), connection is closed, or publishing fails. |

---

### Output Payload Examples

#### 1. Successful Publish (`success`)

When a message is successfully transmitted, the node emits the following confirmation payload:

```json
{
  "success": true,
  "topic": "/topic/chat.room-505",
  "message": "Hello people in room-505!",
  "brokerURL": "wss://broker.example.com/ws"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `success` | `boolean` | Confirms that the message was sent to the broker. |
| `topic` | `string` | The resolved destination topic or queue on which the message was published. |
| `message` | `string` | The exact string payload transmitted in the STOMP frame. |
| `brokerURL` | `string` | The WebSocket endpoint URL of the broker that received the message. |

#### 2. Publishing Failure (`error`)

If a connection failure or timeout occurs during publishing, the node returns:

```json
{
  "success": false,
  "error": "STOMP connection timeout",
  "topic": "/topic/chat.room-505",
  "message": "Hello people in room-505!"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `success` | `boolean` | Value is `false` indicating an unsuccessful attempt. |
| `error` | `string` | Error diagnostic message (e.g. `STOMP connection timeout`, `STOMP connection closed`, or `Topic/destination is required`). |
| `topic` | `string` | The destination topic targeted by the failed attempt. |
| `message` | `string` | The message body that failed to send. |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

The following example workflow demonstrates two publishing approaches:
1. **Dynamic Publishing:** A Manual Trigger feeds a Function node that dynamically computes destination and message strings, which are passed to the **STOMP Publish** node via expressions.
2. **Static Publishing:** A second Manual Trigger fires a **STOMP Publish** node configured with a static destination and JSON payload.

```fusion-workflow
src: example.workflow.json
title: Publish messages to STOMP destination
```

### Complete Workflow Definition

```json
{
    "name": "STOMP Publish1",
    "variables": {},
    "secrets": {},
    "nodes": [
        {
            "position": {
                "x": 437.37762586186864,
                "y": 34.21607060222868
            },
            "width": 256,
            "height": 100,
            "dragHandle": ".drag-handle__custom",
            "selected": false,
            "id": "r24fdkibi9nqk5jbg9pfoyq4",
            "type": "display",
            "data": {
                "description": "Logs the input data to the console.",
                "showRunningStatus": false,
                "_morphing": false,
                "name": "log",
                "label": "Log1",
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
        },
        {
            "position": {
                "x": 250.3776258618687,
                "y": 77.71607060222868
            },
            "width": 72,
            "height": 72,
            "selected": false,
            "id": "ae1k5lvtmm65pee4cgn9nxhr",
            "type": "action",
            "data": {
                "description": "Publish messages to STOMP destinations. Supports WebSocket-based STOMP brokers with authentication.",
                "showRunningStatus": true,
                "_morphing": false,
                "name": "stomp-publish",
                "label": "STOMP Publish",
                "inputs": {
                    "input": {
                        "label": "Input"
                    }
                },
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
                    "brokerURL": "wss://7766-2c0e-6042-40-979f-707e-810c-3656-c8a3.ngrok-free.app/ws",
                    "topic": {
                        "$expr": "output",
                        "node": "Function",
                        "outputId": "success",
                        "path": "topic"
                    },
                    "login": "guest",
                    "passcode": "guest",
                    "message": {
                        "$expr": "output",
                        "node": "Function",
                        "outputId": "success",
                        "path": "message"
                    }
                },
                "defaultOutput": null
            },
            "inputs": null,
            "outputs": null
        },
        {
            "position": {
                "x": 97.75632983048038,
                "y": 30.645944367457332
            },
            "width": 72,
            "height": 72,
            "selected": false,
            "id": "no292swu9b13zid496zympv9",
            "type": "action",
            "data": {
                "description": "Executes custom JavaScript code on the input data.",
                "showRunningStatus": true,
                "_morphing": false,
                "name": "function",
                "label": "Function",
                "inputs": {
                    "input": {
                        "label": "Input"
                    }
                },
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
                    "code": "const roomId = \"room-505\";\nreturn {\n  topic: `/topic/chat.${roomId}`,\n  message: `Hello people in ${roomId}!`\n};"
                },
                "defaultOutput": null
            },
            "inputs": null,
            "outputs": null
        },
        {
            "position": {
                "x": -41.622374138131306,
                "y": 77.71607060222868
            },
            "width": 72,
            "height": 72,
            "selected": false,
            "id": "o7ib440y2jzg025jtc2gunp7",
            "type": "trigger",
            "data": {
                "description": "Triggers the workflow manually.",
                "showRunningStatus": false,
                "_morphing": false,
                "name": "manual-trigger",
                "label": "Manual Trigger",
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
                "parameters": {},
                "defaultOutput": null
            },
            "inputs": null,
            "outputs": null
        },
        {
            "position": {
                "x": 316.86078371025064,
                "y": 193.80364327389265
            },
            "width": 256,
            "height": 100,
            "dragHandle": ".drag-handle__custom",
            "selected": false,
            "id": "ljukzz9v9mg61n7bc4e4sivo",
            "type": "display",
            "data": {
                "description": "Logs the input data to the console.",
                "showRunningStatus": false,
                "_morphing": false,
                "name": "log",
                "label": "Log2",
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
        },
        {
            "position": {
                "x": 141.8607837102507,
                "y": 249.30364327389265
            },
            "width": 72,
            "height": 72,
            "selected": true,
            "id": "ma0w1bujnjnexrhuot01j8rf",
            "type": "action",
            "data": {
                "description": "Publish messages to STOMP destinations. Supports WebSocket-based STOMP brokers with authentication.",
                "showRunningStatus": true,
                "_morphing": false,
                "name": "stomp-publish",
                "label": "STOMP Publish1",
                "inputs": {
                    "input": {
                        "label": "Input"
                    }
                },
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
                    "brokerURL": "wss://7766-2c0e-6042-40-979f-707e-810c-3656-c8a3.ngrok-free.app/ws",
                    "topic": "/topic/cloud-test",
                    "login": "guest",
                    "passcode": "guest",
                    "message": "{\"message\": \"salam STOMP \", \"test\": true}"
                },
                "defaultOutput": null
            },
            "inputs": null,
            "outputs": null
        },
        {
            "position": {
                "x": -33.1392162897493,
                "y": 241.30364327389265
            },
            "width": 72,
            "height": 72,
            "selected": false,
            "id": "r6w80w6nhhyeyl9a97kn22iv",
            "type": "trigger",
            "data": {
                "description": "Triggers the workflow manually.",
                "showRunningStatus": false,
                "_morphing": false,
                "name": "manual-trigger",
                "label": "Manual Trigger1",
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
            "id": "xy-edge__ae1k5lvtmm65pee4cgn9nxhrsuccess-r24fdkibi9nqk5jbg9pfoyq4input",
            "data": {
                "isAnimated": false
            },
            "source": "ae1k5lvtmm65pee4cgn9nxhr",
            "target": "r24fdkibi9nqk5jbg9pfoyq4",
            "sourceHandle": "success",
            "targetHandle": "input"
        },
        {
            "type": "directed",
            "id": "xy-edge__ae1k5lvtmm65pee4cgn9nxhrerror-r24fdkibi9nqk5jbg9pfoyq4input",
            "data": {
                "isAnimated": false
            },
            "source": "ae1k5lvtmm65pee4cgn9nxhr",
            "target": "r24fdkibi9nqk5jbg9pfoyq4",
            "sourceHandle": "error",
            "targetHandle": "input"
        },
        {
            "type": "directed",
            "id": "xy-edge__no292swu9b13zid496zympv9success-ae1k5lvtmm65pee4cgn9nxhrinput",
            "data": {
                "isAnimated": false
            },
            "source": "no292swu9b13zid496zympv9",
            "target": "ae1k5lvtmm65pee4cgn9nxhr",
            "sourceHandle": "success",
            "targetHandle": "input"
        },
        {
            "type": "directed",
            "id": "xy-edge__o7ib440y2jzg025jtc2gunp7success-no292swu9b13zid496zympv9input",
            "data": {
                "isAnimated": false
            },
            "source": "o7ib440y2jzg025jtc2gunp7",
            "target": "no292swu9b13zid496zympv9",
            "sourceHandle": "success",
            "targetHandle": "input"
        },
        {
            "type": "directed",
            "id": "zij2nsca0dhtznzl273k9zmh",
            "data": {
                "isAnimated": false
            },
            "source": "ma0w1bujnjnexrhuot01j8rf",
            "target": "ljukzz9v9mg61n7bc4e4sivo",
            "sourceHandle": "success",
            "targetHandle": "input"
        },
        {
            "type": "directed",
            "id": "pqsubfm31sx6xa0gjour7eva",
            "data": {
                "isAnimated": false
            },
            "source": "ma0w1bujnjnexrhuot01j8rf",
            "target": "ljukzz9v9mg61n7bc4e4sivo",
            "sourceHandle": "error",
            "targetHandle": "input"
        },
        {
            "id": "r6w80w6nhhyeyl9a97kn22iv-ma0w1bujnjnexrhuot01j8rf-4f3e5c43-5dd3-451b-8710-ec6f91282991",
            "type": "directed",
            "animated": false,
            "data": {
                "isAnimated": false
            },
            "source": "r6w80w6nhhyeyl9a97kn22iv",
            "target": "ma0w1bujnjnexrhuot01j8rf",
            "sourceHandle": "success",
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
    "updatedAt": 1790846681498
}
```

---

### Practical Architecture Patterns

#### Pattern 1: Dynamic Chat Room Broadcast

1. **Webhook / API Trigger:** Receives an incoming chat message payload `{ "roomId": "505", "text": "Hello team!" }`.
2. **Function Node:** Formats the destination topic and payload:
   ```javascript
   return {
     topic: `/topic/chat.room-${input.roomId}`,
     message: JSON.stringify({
       sender: input.sender || "System",
       text: input.text,
       sentAt: new Date().toISOString()
     })
   };
   ```
3. **STOMP Publish Node:** Publishes the message dynamically using the returned `topic` and `message`.
4. **Log Node:** Records the confirmation message and broker status.

#### Pattern 2: E-Commerce Microservice Event Bus

1. **Database / Form Trigger:** Triggers when a new customer checkout completes.
2. **STOMP Publish Node:** Broadcasts the checkout event to RabbitMQ Web STOMP at `/exchange/orders.topic/order.created`.
3. **Downstream Systems:** Subscribed inventory, billing, and fulfillment microservices immediately consume the event concurrently.

#### Pattern 3: Real-Time Dashboard KPI Stream

1. **Interval Trigger:** Runs every 10 seconds to collect real-time server health and active user metrics.
2. **Function Node:** Compiles current metrics into a JSON object.
3. **STOMP Publish Node:** Publishes to `/topic/metrics.system` on `wss://broker.example.com/ws`. Connected web dashboards automatically update graphs in real time.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Topic/destination is required`

- **Cause:** Neither `topic` in the node configuration nor `topic` in the incoming data payload was provided.
- **Solution:** Specify a default destination topic in the node configuration (e.g. `/topic/general`), or ensure upstream nodes pass a `topic` field in their output.

#### `Message is required`

- **Cause:** Neither `message` was configured in the node panel nor was any valid message string or data object provided by upstream nodes.
- **Solution:** Supply a message string in the configuration or pass data from an upstream node (such as a Function node).

#### `STOMP connection timeout`

- **Cause:** The node attempted to connect to `brokerURL`, but the broker did not complete the WebSocket handshake within the 10-second timeout.
- **Solution:**
  1. Confirm the broker host is running and the WebSocket port is reachable.
  2. Verify your WebSocket protocol: use `wss://` for SSL/TLS connections and `ws://` for plain connections.
  3. Ensure reverse proxies (like Nginx, Traefik, or AWS ALB) allow WebSocket connections and do not drop the `Upgrade: websocket` header.

#### `STOMP connection closed`

- **Cause:** The broker actively rejected or immediately severed the connection upon opening.
- **Solution:**
  1. Check broker authentication: verify that `login` and `passcode` are correct.
  2. Check broker logs for permission rejections (e.g. user lacks write permission to the requested destination).
  3. Verify that the STOMP over WebSocket plugin is enabled on your broker (e.g. `rabbitmq_web_stomp` for RabbitMQ).

#### Destination not receiving messages on subscriber clients

- **Cause:** Destination formatting mismatch between publisher and subscriber.
- **Solution:** Ensure the topic string matches the exact case and broker prefix (e.g. RabbitMQ requires `/topic/` for topic exchanges; publishing to `chat` without `/topic/` may cause RabbitMQ to reject or misroute the message).

---

### Error Reference

| Error Indicator | Possible Cause | Recommended Resolution |
|-----------------|----------------|------------------------|
| `Topic/destination is required` | No destination specified in config or payload | Provide a valid destination string. |
| `Message is required` | No message provided and input data is empty | Provide a message body in config or upstream data. |
| `STOMP connection timeout` | Broker unreachable within 10 seconds | Check broker availability, network firewall, and port accessibility. |
| `STOMP connection closed` | Connection rejected or severed by broker | Check broker logs and authentication credentials. |
| `Invalid URL` | `brokerURL` is not a valid WebSocket URL | Ensure URL starts with `ws://` or `wss://`. |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [STOMP Subscribe](./stomp-subscribe.md) — Subscribe to STOMP destinations and trigger workflows on incoming messages
- [NATS Publish](./nats-publish.md) — Publish messages to NATS subjects
- [Function](./function.md) — Construct and transform dynamic payload messages
- [Manual Trigger](./manual-trigger.md) — Manually trigger workflows for testing
- [Log](./log.md) — Inspect publishing confirmations and execution logs

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-10-01 | Initial release of the STOMP Publish action node with WebSocket connectivity, dynamic topic/message routing, automatic JSON serialization, and broker authentication. |

<!-- /SECTION: changelog -->
