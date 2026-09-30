---
node_id: "nats-publish"
title: "NATS Publish"
description: "Publish messages to NATS subjects with support for custom headers, dynamic runtime payloads, reply subjects, and synchronous request-reply waiting."
category: "communication"
subcategory: "queues-event-streams"
version: "1.0.0"
language: "en"
last_updated: "2026-09-30"
author: "Fusion Team"
tags:
  - nats
  - publish
  - messaging
  - pubsub
  - event-driven
  - request-reply
  - queues
related_nodes:
  - nats-subscribe
  - kafka-publish
  - mqtt-publisher
  - function
  - manual-trigger
---

<!-- SECTION: overview -->
# NATS Publish

> **Category:** Communication & Messaging&nbsp;&nbsp;|&nbsp;&nbsp;**Subgroup:** Queues & Event Streams&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

The **NATS Publish** node produces and sends messages to target subjects in a [NATS](https://nats.io/) messaging system. It acts as an event emitter or client within distributed architectures, supporting automatic JSON serialization, message headers, dynamic runtime routing, and two-way **Request-Reply** communication patterns with reply subject listeners.

### Key Features

- **Automatic Payload Serialization:** Automatically serializes incoming workflow data (objects, arrays, strings) into JSON text before publishing to the NATS cluster.
- **Dynamic Routing Overrides:** Allows subject, message body, headers, and reply parameters to be set either statically in the node configuration or dynamically from incoming upstream data.
- **Request-Reply Support:** Supports publishing with a `reply` subject and automatically waits for the recipient's response, bringing synchronous RPC capabilities to asynchronous workflows.
- **Custom Message Headers:** Attach metadata, tracing IDs, and content-type headers as key-value pairs to published messages.
- **Immediate Buffer Flush:** Configurable `flushAfterPublish` option to immediately flush client buffers and ensure delivery confirmation before downstream workflow steps execute.
- **Clustered & Secure Connections:** Supports single servers or multi-node clusters with anonymous, username/password, or secret token authentication.
- **Connection Reuse:** Automatically manages client connection lifecycles across workflow ticks for optimal throughput.

### Use Cases

- **Asynchronous Event Broadcasting:** Broadcast events (e.g. order placed, user created, file uploaded) to multiple downstream microservices subscribing to NATS topics.
- **Request-Reply RPC Calls:** Send queries to backend services over NATS and capture the response directly in the workflow output.
- **Responding to Inbound Subscriptions:** Pair with [NATS Subscribe](./nats-subscribe.md) to process inbound requests and reply back to the caller's `reply` inbox.
- **Telemetry & Metric Publishing:** Publish sensor readings, workflow audit logs, or operational metrics into central monitoring streams.
- **Dynamic Multi-Tenant Ingress:** Route notifications or tenant-specific events dynamically using payload values like `{{input.tenantId}}.events`.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

Configure broker connection endpoints, authentication credentials, and default publishing options in the node configuration panel.

### Connection Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `servers` | `string \| string[]` | ✅ Yes | — | NATS server address(es). Accepts a single URL string, a comma-separated list of URLs, or an array of URLs (e.g. `nats://127.0.0.1:4222` or `["nats://srv1:4222", "nats://srv2:4222"]`). |

### Authentication Parameters (Optional)

Configure credentials if your NATS cluster enforces client authentication:

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `username` | `string` | ❌ No | — | Username for basic client authentication. |
| `password` | `string` | ❌ No | — | Password for basic client authentication. |
| `token` | `string` | ❌ No | — | Secret authentication token (alternative to username/password). |

### Message Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `subject` | `string` | ✅ Yes* | — | Target NATS subject to publish to (e.g. `orders.created`, `notifications.email`). Can be overridden dynamically by `input.subject`. |
| `message` | `string` | ❌ No* | — | Static message payload to publish. If left empty, the node automatically serializes the incoming workflow `input` data to JSON. |
| `headers` | `string` | ❌ No | — | Optional JSON string of key-value header pairs (e.g. `{"source": "workflow", "traceId": "123"}`). Can also be provided as an object via `input.headers`. |
| `reply` | `string` | ❌ No | — | Reply subject address. When provided, the node publishes with this reply subject, flushes, and waits for 1 incoming reply message. |
| `flushAfterPublish` | `boolean` | ❌ No | `false` | When `true`, flushes the connection buffer immediately after sending to guarantee transmission before continuing. |

> *\* **Dynamic Flexibility:** If `subject` or `message` is not defined statically in the configuration panel, the node expects them to be provided in the incoming `input` data (`input.subject` or `input.message`). If no explicit message is defined, the entire `input` object is serialized and published as JSON.*

---

### Dynamic Payload Overrides

The **NATS Publish** node supports dynamic runtime evaluation. Any property provided in the upstream input object overrides the corresponding static configuration:

```json
{
  "subject": "orders.eu.created",
  "message": "Custom message or object",
  "headers": {
    "correlation-id": "corr-9921",
    "priority": "high"
  },
  "reply": "_INBOX.temp.123",
  "flushAfterPublish": true
}
```

If the upstream node simply sends a standard data object without specific NATS wrapper keys, the **NATS Publish** node automatically stringifies the entire object to JSON and publishes it to the configured `subject`.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Upstream data stream. Can be an arbitrary JSON object, string, or an object containing override properties (`subject`, `message`, `headers`, `reply`, `flushAfterPublish`). |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted when the message is successfully published (or when a reply message is received in Request-Reply mode). |
| `error` | `object` | Emitted if connection fails, publishing encounters an error, or payload validation fails. |

---

### Output Schema (`success`)

#### Standard Publish Mode (No Reply Subject)

When publishing without a reply subject, the node emits confirmation of delivery along with the published metadata:

```json
{
  "success": true,
  "subject": "orders.created",
  "message": "{\"orderId\":\"ORD-8912\",\"total\":149.99}",
  "headers": {
    "source": "fusion-automation",
    "trace-id": "trace-44b-81a"
  },
  "reply": null
}
```

| Field | Type | Description |
|-------|------|-------------|
| `success` | `boolean` | `true` indicating the message was successfully dispatched to NATS. |
| `subject` | `string` | The NATS subject to which the message was published. |
| `message` | `string` | The message payload that was sent. |
| `headers` | `object \| null` | The headers attached to the message, or `null` if none were specified. |
| `reply` | `string \| null` | `null` when no reply subject was requested. |

---

#### Request-Reply Mode (With Reply Subject)

When `reply` is specified, the node publishes the request and waits for the first response on the reply inbox:

```json
{
  "success": true,
  "subject": "inventory.check",
  "replySubject": "_INBOX.8821ad.1",
  "message": "{\"productId\":\"SKU-9921\"}",
  "reply": "{\"available\":true,\"stock\":42}"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `success` | `boolean` | `true` indicating the request was sent and a reply was received. |
| `subject` | `string` | The original target subject of the request. |
| `replySubject` | `string` | The temporary inbox subject where the response was expected. |
| `message` | `string` | The outgoing request payload. |
| `reply` | `string` | The raw message body received from the answering service. |

---

### Output Schema (`error`)

If an error occurs while connecting, serializing, or publishing, an error object is emitted:

```json
{
  "success": false,
  "error": "Failed to connect to NATS: connect ECONNREFUSED 127.0.0.1:4222",
  "subject": "orders.created",
  "message": "{\"orderId\":\"ORD-8912\"}"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `success` | `boolean` | `false` indicating operation failure. |
| `error` | `string` | Detailed message explaining the failure reason. |
| `subject` | `string` | The target subject that was being published to. |
| `message` | `string` | The payload that failed to send. |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

The following example demonstrates triggering a workflow via a Manual Trigger, publishing the message payload to a NATS subject, and logging the output:

```fusion-workflow
src: example.workflow.json
title: Publish a message to NATS on manual trigger
```

### Complete Workflow Definition

```json
{
  "name": "NATS Publish",
  "nodes": [
    {
      "id": "manual-trigger",
      "type": "trigger",
      "position": {
        "x": 80,
        "y": 120
      },
      "width": 64,
      "height": 64,
      "data": {
        "name": "manual-trigger",
        "label": "Manual Trigger",
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
      "id": "nats-publish",
      "type": "action",
      "position": {
        "x": 280,
        "y": 120
      },
      "width": 64,
      "height": 64,
      "data": {
        "name": "nats-publish",
        "label": "NATS Publish",
        "inputs": {
          "input": {
            "label": "Input"
          }
        },
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
        "x": 480,
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
      "id": "xy-edge__manual-triggersuccess-nats-publishinput",
      "type": "directed",
      "source": "manual-trigger",
      "target": "nats-publish",
      "sourceHandle": "success",
      "targetHandle": "input"
    },
    {
      "id": "xy-edge__nats-publishsuccess-loginput",
      "type": "directed",
      "source": "nats-publish",
      "target": "log",
      "sourceHandle": "success",
      "targetHandle": "input"
    }
  ]
}
```

---

### Architecture Patterns

#### Pattern 1: Webhook to NATS Event Pipeline

Receive incoming HTTP webhooks, normalize the payload using a Function node, and publish the event into an internal microservice NATS topic:

1. **Webhook Trigger:** Ingests an external payment event from Stripe or PayPal.
2. **Function Node:** Cleans and formats the event into an enterprise schema:
   ```javascript
   return {
     subject: `payments.${input.currency.toLowerCase()}.processed`,
     message: JSON.stringify(input)
   };
   ```
3. **NATS Publish:** Emits the event directly to the dynamically computed subject.

#### Pattern 2: Responding to a NATS Subscription (Request-Reply Service)

Pair [NATS Subscribe](./nats-subscribe.md) and **NATS Publish** to implement an RPC service:

1. **NATS Subscribe:** Listens to `service.customer.lookup`.
2. **Database Query Node:** Fetches customer profile information by ID.
3. **NATS Publish Node:** Publishes the customer details back to the caller:
   - `subject`: Configured dynamically as `{{input.reply}}`
   - `message`: `{{input.queryResult}}`

#### Pattern 3: Dead-Letter Queue (DLQ) Notification

When an action node in a critical workflow encounters an error, route the failure context to a monitoring NATS subject:

1. **Action Node:** Emits an event on its `error` handle.
2. **NATS Publish:** Sends the error object to `system.errors.critical` with custom headers (`{"severity": "P1"}`).

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `At least one NATS server URL is required`

- **Cause:** The `servers` parameter is empty or blank in the configuration.
- **Solution:** Provide one or more valid NATS connection URLs, such as `nats://localhost:4222` or `nats://node1:4222,nats://node2:4222`.

#### `Failed to connect to NATS: connect ECONNREFUSED`

- **Cause:** The NATS server is offline, the port number is incorrect, or a firewall is blocking TCP communication on port `4222`.
- **Solution:** Verify that the NATS server is running and accessible from the workflow host using `nc -zv <host> 4222`.

#### `Subject is required`

- **Cause:** No `subject` was specified in the configuration panel, and the incoming payload did not supply `input.subject`.
- **Solution:** Specify a static subject in the node configuration or pass a non-empty `subject` property in the incoming data.

#### `Invalid JSON in headers field`

- **Cause:** The `headers` parameter was provided as a string that does not conform to valid JSON format.
- **Solution:** Ensure the headers string is well-formed JSON (e.g. `{"correlation-id": "123"}`), or pass `headers` directly as an object in the input data.

#### Workflow hangs when using Request-Reply mode

- **Cause:** A `reply` subject was specified, but no external NATS subscriber answered the request on that subject.
- **Solution:** Ensure the target service receiving the request is active and programmed to publish its response to the specified `reply` inbox.

---

### Error Reference

| Error Indicator | Possible Cause | Recommended Resolution |
|-----------------|----------------|------------------------|
| `At least one NATS server URL is required` | Missing `servers` field | Specify at least one valid server URL. |
| `Subject is required` | Missing subject name | Set `subject` in config or pass `input.subject`. |
| `Message is required` | Empty input and no message configured | Provide a static `message` or pass data into the `input` port. |
| `Invalid JSON in headers field` | Malformed JSON in headers parameter | Verify JSON syntax for headers. |
| `ECONNREFUSED` | Broker unreachable | Check host, port, and network route. |
| `Authorization Violation` | Authentication failed | Check `username`, `password`, or `token` credentials. |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [NATS Subscribe](./nats-subscribe.md) — Subscribe to NATS subjects and trigger workflows on incoming messages
- [Kafka Publish](./kafka-publish.md) — Publish messages to Apache Kafka topics
- [MQTT Publisher](./mqtt-publisher.md) — Publish MQTT messages to IoT topics
- [Function](./function.md) — Prepare, validate, and construct message payloads
- [Manual Trigger](./manual-trigger.md) — Manually trigger workflow execution for testing

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-30 | Initial release with support for dynamic message payloads, custom headers, flush control, and request-reply mode. |

<!-- /SECTION: changelog -->
