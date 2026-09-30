---
node_id: "nats-subscribe"
title: "NATS Subscribe"
description: "Subscribe to NATS subjects and trigger workflow executions on incoming messages, supporting wildcard matching, authentication, and message count limits."
category: "triggers"
subcategory: "messaging"
version: "1.0.0"
language: "en"
last_updated: "2026-09-30"
author: "Fusion Team"
tags:
  - nats
  - subscribe
  - trigger
  - messaging
  - pubsub
  - event-driven
  - real-time
related_nodes:
  - nats-publish
  - kafka-trigger
  - mqtt-subscriber
  - function
  - log
---

<!-- SECTION: overview -->
# NATS Subscribe

> **Category:** Triggers & Ingress&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Trigger Node

The **NATS Subscribe** node connects to a [NATS](https://nats.io/) messaging system and triggers workflow executions whenever a message is published to a subscribed subject. It supports single or clustered server configurations, wildcard subject matching, authentication (user/password or token), automatic JSON decoding, and request-reply integration via reply subjects.

### Key Features

- **Clustered & Standalone Connections:** Connect to a single NATS server or provide a list of cluster nodes for resilient failover.
- **Hierarchical & Wildcard Subscriptions:** Listen on exact subject names or leverage NATS single-token (`*`) and full-hierarchy (`>`) wildcards.
- **Automatic JSON Parsing:** Incoming binary UTF-8 messages are automatically parsed as JSON into `parsedData` with a graceful fallback to raw text in `data`.
- **Request-Reply Ready:** Captures incoming `reply` subjects so workflows can process queries and return responses using a downstream publisher.
- **Header Propagation:** Extracts all NATS message headers and exposes them as structured key-value pairs.
- **Message Limit Control (`max`):** Optionally configure a message threshold to auto-unsubscribe after processing a designated count of messages.
- **Flexible Authentication:** Supports anonymous connections, standard username/password credentials, or token-based authentication.
- **Workflow Lifecycle Control:** Seamlessly pause and resume message consumption without losing configuration.

### Use Cases

- **Event-Driven Workflows:** Trigger asynchronous business logic whenever microservices emit events (e.g. order placed, user registered).
- **IoT & Telemetry Processing:** Collect sensor measurements across multi-level wildcard subjects like `sensors.factory1.*.temperature`.
- **Request-Reply Microservice Endpoints:** Build lightweight RPC endpoints that process requests and respond back to the caller using the captured `reply` subject.
- **Real-Time Data Distribution:** Ingest streaming messages from messaging topologies and route them to databases, notification services, or dashboards.
- **Controlled Ingestion:** Consume a precise batch of messages (`max: 50`) for scheduled synchronization or testing workflows.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

Configure connection endpoints, authentication credentials, and subscription rules in the node configuration panel.

### Connection Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `servers` | `string \| string[]` | ✅ Yes | — | NATS server connection URL(s). Supports a single URL, a comma-separated list of URLs, or an array of URLs (e.g. `nats://127.0.0.1:4222` or `["nats://node1:4222", "nats://node2:4222"]`). |

### Authentication Parameters (Optional)

Configure authentication credentials if your NATS server requires client verification:

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `username` | `string` | ❌ No | — | Username for basic authentication. |
| `password` | `string` | ❌ No | — | Password for basic authentication. |
| `token` | `string` | ❌ No | — | Secret authentication token (alternative to username/password). |

> **Note:** If both username/password and token are supplied, the node includes both in the connection options. Ensure only the authentication method expected by your NATS server is configured.

### Subscription Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `subject` | `string` | ✅ Yes | — | The NATS subject name or pattern to subscribe to (e.g. `orders.created` or `telemetry.*.temperature`). |
| `max` | `number` | ❌ No | — | Maximum number of messages to receive before automatically stopping the subscription. |
| `timeout` | `number` | ❌ No | — | Timeout duration in milliseconds for operations. |

---

### Working with NATS Subjects and Wildcards

NATS subjects are dot-separated tokens (e.g. `eu.orders.b2b.created`). The **NATS Subscribe** node supports full NATS wildcard syntax:

| Wildcard | Name | Scope | Example | Matches | Does Not Match |
|----------|------|-------|---------|---------|----------------|
| `*` | Single-token | Matches exactly one token between dots | `orders.*.created` | `orders.eu.created`, `orders.us.created` | `orders.created`, `orders.eu.retail.created` |
| `>` | Multi-token tail | Matches one or more tokens at the end of a subject | `orders.>` | `orders.created`, `orders.eu.created`, `orders.eu.b2b.completed` | `billing.created` |

> **Tip:** The node's output payload includes the concrete `subject` on which each message was published. When listening to wildcards like `telemetry.*.status`, downstream nodes can read `{{input.subject}}` to identify the originating device.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

This node is a **Trigger Node**. It initiates workflow executions upon receiving messages from the NATS broker and does not accept incoming connections from upstream nodes.

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted whenever a message is received on the subscribed subject. |
| `error` | `object` | Emitted when connection fails, authentication is rejected, or an internal subscription error occurs. |

---

### Output Schema (`success`)

When a message arrives, the node decodes the binary body, attempts JSON parsing, and emits the following structured object:

```json
{
  "subject": "orders.eu.created",
  "data": "{\"orderId\":\"ORD-4512\",\"customer\":\"Alpha Inc\",\"total\":249.99}",
  "parsedData": {
    "orderId": "ORD-4512",
    "customer": "Alpha Inc",
    "total": 249.99
  },
  "headers": {
    "Nats-Msg-Id": "msg-882103",
    "correlation-id": "trace-77a-b9c"
  },
  "reply": "_INBOX.9u8y7t6r5e",
  "timestamp": "2026-09-30T15:35:10.124Z"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `subject` | `string` | The exact subject on which the message was published. |
| `data` | `string` | Raw message body decoded from UTF-8 bytes into a string. |
| `parsedData` | `any` | Automatically parsed JSON object or array. If the message payload is plain text or invalid JSON, it falls back safely to the raw string. |
| `headers` | `object \| undefined` | Key-value dictionary of message headers if headers were present; otherwise `undefined`. |
| `reply` | `string \| undefined` | Reply inbox subject address provided by the publisher (present when request-reply is used); otherwise `undefined`. |
| `timestamp` | `string` | ISO 8601 timestamp string representing the moment the node received and processed the message. |

---

### Output Schema (`error`)

If the connection fails or an error occurs during message handling, an event is emitted on the `error` port:

```json
{
  "error": "Failed to connect to NATS: connect ECONNREFUSED 127.0.0.1:4222"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `error` | `string` | Detailed error message describing why the connection or subscription failed. |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

The following example workflow demonstrates listening for incoming NATS messages and immediately displaying the incoming payload in a Log display node:

```fusion-workflow
src: example.workflow.json
title: Subscribe to NATS messages and log payload
```

### Complete Workflow Definition

```json
{
  "name": "NATS Subscribe to Log",
  "nodes": [
    {
      "id": "nats-subscribe",
      "type": "trigger",
      "position": {
        "x": 80,
        "y": 120
      },
      "width": 64,
      "height": 64,
      "data": {
        "name": "nats-subscribe",
        "label": "NATS Subscribe",
        "inputs": {},
        "outputs": {
          "success": {
            "label": "Success"
          },
          "error": {
            "label": "Error"
          }
        }
      }
    },
    {
      "id": "log",
      "type": "display",
      "position": {
        "x": 320,
        "y": 100
      },
      "width": 256,
      "height": 100,
      "data": {
        "name": "log",
        "label": "Log",
        "inputs": {
          "input": {
            "label": "Input"
          }
        },
        "outputs": {
          "success": {
            "label": "Success"
          }
        }
      }
    }
  ],
  "connections": [
    {
      "id": "xy-edge__nats-subscribesuccess-loginput",
      "type": "directed",
      "source": "nats-subscribe",
      "target": "log",
      "sourceHandle": "success",
      "targetHandle": "input"
    }
  ]
}
```

---

### Architecture Patterns

#### Pattern 1: Order Ingestion Pipeline

1. **NATS Subscribe:** Listens to subject `orders.*.created`.
2. **Function Node:** Validates required properties (e.g. `input.parsedData.orderId` and `amount`).
3. **Database Action Node:** Inserts the validated order record into PostgreSQL or MongoDB.
4. **Notification Node:** Sends a notification message (Slack/Email) if the order total exceeds a specified threshold.

#### Pattern 2: Request-Reply Microservice Responder

1. **NATS Subscribe:** Listens on `inventory.query`.
2. **Database Query Node:** Checks product availability based on `{{input.parsedData.productId}}`.
3. **NATS Publish Node:** Publishes the resulting inventory status directly to the subject specified in `{{input.reply}}`, answering the client synchronously.

#### Pattern 3: Wildcard Telemetry Router

1. **NATS Subscribe:** Listens to `telemetry.sensors.>`.
2. **Function Node:** Parses temperature readings and reads the sensor location from the subject segments.
3. **Filter Node:** Discards normal readings and forwards only critical threshold exceedances to an incident management system.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `At least one NATS server URL is required`

- **Cause:** The `servers` parameter is empty or contains only whitespace.
- **Solution:** Specify one or more valid server URLs, such as `nats://localhost:4222` or `nats://broker.example.internal:4222`.

#### `Failed to connect to NATS: connect ECONNREFUSED`

- **Cause:** The NATS server is not running, the hostname/IP is incorrect, or a firewall blocks TCP port `4222`.
- **Solution:** Verify the NATS service is active and accessible from the workflow runtime host using `telnet` or `nc -zv <host> 4222`.

#### Node starts but triggers never fire

- **Cause:** The subscribed `subject` does not match the subject used by the publisher, or publishers are inactive.
- **Solution:** 
  1. Confirm the subject name spelling.
  2. If using wildcards, remember that `*` only matches a single token (e.g. `orders.*` will **not** match `orders.eu.created`; use `orders.>` or `orders.*.*` instead).
  3. Verify message delivery on the subject using the NATS CLI: `nats sub "<subject>"`.

#### Subscription stops receiving messages after a certain number of events

- **Cause:** The `max` parameter is set to a specific number, causing the subscription to automatically terminate once that message count is reached.
- **Solution:** Remove or clear the `max` field if you want the trigger to listen continuously.

#### `parsedData` contains a string instead of an object

- **Cause:** The message payload published to NATS is plain text, CSV, or invalid JSON.
- **Solution:** If the publisher is sending JSON, ensure it produces well-formed JSON syntax. If plain text or binary is intended, use downstream nodes (such as a Function node) to process the raw string from `data`.

#### `Authorization Violation` error

- **Cause:** The NATS server requires authentication, but credentials were not provided, or invalid username/password/token was supplied.
- **Solution:** Check NATS server access control configuration and provide valid credentials in the node's `username`, `password`, or `token` fields.

---

### Error Reference

| Error Indicator | Possible Cause | Recommended Resolution |
|-----------------|----------------|------------------------|
| `At least one NATS server URL is required` | Empty `servers` configuration | Provide at least one valid server URL. |
| `ECONNREFUSED` | NATS daemon not reachable on the specified port | Verify server hostname, port (default `4222`), and firewall rules. |
| `Authorization Violation` | Authentication failed or missing credentials | Verify `username`, `password`, or `token` match server permissions. |
| `Permissions Violation` | User account does not have read permissions for the subject | Check subject authorization rules in the NATS server configuration (`nats.conf`). |
| `Slow Consumer` | Downstream processing backlog | Ensure downstream nodes process events efficiently or scale workflow workers. |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [NATS Publish](./nats-publish.md) — Publish messages to NATS subjects
- [Kafka Trigger](./kafka-trigger.md) — Trigger workflows on Apache Kafka topic events
- [MQTT Subscriber](./mqtt-subscriber.md) — Ingest messages from MQTT brokers
- [Function](./function.md) — Transform, validate, and enrich incoming message payloads
- [Log](./log.md) — Inspect workflow outputs in the execution log

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-30 | Initial release with support for subject wildcards, auto-JSON decoding, request-reply `reply` subject extraction, and authentication. |

<!-- /SECTION: changelog -->
