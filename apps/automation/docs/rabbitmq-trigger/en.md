---
node_id: "rabbitmq-trigger"
title: "RabbitMQ Trigger"
description: "Trigger workflow execution when messages arrive in a RabbitMQ queue — supports prefetch, manual ack, and auto queue creation"
category: "triggers"
subcategory: "rabbitmq"
version: "1.0.0"
language: "en"
last_updated: "2026-03-10"
author: "Fusion Team"
tags:
  - rabbitmq
  - trigger
  - messaging
  - amqp
  - consumer
  - real-time
related_nodes:
  - rabbitmq-action
  - kafka-trigger
  - redis-subscribe
---

<!-- SECTION: header -->
# RabbitMQ Trigger

> **Category:** Triggers | **Type:** Trigger Node

Subscribe to a RabbitMQ queue and trigger a workflow execution for every incoming message. Supports prefetch rate limiting, manual acknowledgment, and automatic queue creation.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **RabbitMQ Trigger** node connects to a RabbitMQ broker as an AMQP consumer and listens to a named queue. Each incoming message triggers a workflow execution, passing the full message payload including content, AMQP properties, routing fields, and a reception timestamp.

The node supports manual acknowledgment (`noAck: false`) which ensures that messages are only removed from the queue after successful processing. On processing error, the message is negatively acknowledged (`nack`) and discarded — it will not be requeued.

### Key Features

- **Single Queue Consumer:** Subscribe to one queue per node instance
- **Prefetch Rate Limiting:** Control how many unacknowledged messages the broker sends at once
- **Manual Acknowledgment:** Acknowledge or nack messages explicitly after processing
- **Auto Queue Creation:** Optionally declare the queue on startup if it does not exist
- **Durable Queue:** Configurable durability for auto-created queues
- **JSON Auto-Parsing:** Message content is parsed as JSON when possible, otherwise delivered as a raw string
- **Pause / Resume:** Cancel and restart the consumer without disconnecting from the broker

### Use Cases

- Process orders, events, or jobs dispatched by backend services
- Build reliable worker workflows with manual ack to avoid message loss
- React to messages published by the [RabbitMQ Action](./rabbitmq-action.md) node
- Implement rate-limited processing pipelines using `prefetch`
- Fan-in multiple producer services into a single workflow consumer

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Connection Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `host` | `string` | ✅ Yes | — | RabbitMQ broker hostname or IP address |
| `port` | `number` | ✅ Yes | `5672` | AMQP port (use `5671` for TLS) |
| `username` | `string` | ✅ Yes | — | RabbitMQ username |
| `password` | `string` | ✅ Yes | — | RabbitMQ password |
| `vhost` | `string` | ❌ No | `/` | Virtual host to connect to |

### Consumer Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `queue` | `string` | ✅ Yes | — | Name of the queue to consume from |
| `prefetch` | `number` | ❌ No | `1` | Max number of unacknowledged messages to receive at once |
| `noAck` | `boolean` | ❌ No | `false` | Automatically acknowledge messages without waiting for workflow completion |
| `autoCreateQueue` | `boolean` | ❌ No | `true` | Declare the queue on startup if it does not already exist |
| `durable` | `boolean` | ❌ No | `true` | Make the auto-created queue durable |

### `prefetch` Behavior

| Value | Description |
|-------|-------------|
| `1` | Process one message at a time — safest for sequential workflows |
| `> 1` | Allow parallel message processing up to the set limit |

> **Best practice:** Keep `prefetch: 1` unless your workflow is designed for concurrent execution. Higher values improve throughput but require idempotent workflows.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

This is a trigger node — it activates automatically when a message arrives on the subscribed queue. It does not accept inputs from other nodes.

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted for each received message |
| `error` | `Error` | Emitted on processing errors (message is nacked and discarded) |

### Output Schema (`success`)

```json
{
  "queue": "orders",
  "message": {
    "orderId": "4521",
    "customerId": "usr-99",
    "amount": 149.99,
    "status": "created"
  },
  "properties": {
    "contentType": "application/json",
    "deliveryMode": 2,
    "headers": {},
    "messageId": "msg-abc-123"
  },
  "fields": {
    "deliveryTag": 1,
    "exchange": "events",
    "routingKey": "order.created",
    "redelivered": false
  },
  "timestamp": 1741600000123
}
```

| Field | Type | Description |
|-------|------|-------------|
| `queue` | `string` | Name of the queue the message was consumed from |
| `message` | `any` | Message content — auto-parsed as JSON if valid, otherwise raw string |
| `properties` | `object` | AMQP message properties (`contentType`, `deliveryMode`, `headers`, `messageId`, etc.) |
| `fields` | `object` | AMQP delivery fields (`deliveryTag`, `exchange`, `routingKey`, `redelivered`) |
| `timestamp` | `number` | Unix timestamp (ms) when the message was received by the node |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example: Basic Consumer — Process One at a Time

Consume from the `orders` queue with manual ack and prefetch of 1.

**Configuration:**
```json
{
  "host": "rabbitmq.example.com",
  "port": 5672,
  "username": "fusion",
  "password": "secret",
  "vhost": "/",
  "queue": "orders",
  "prefetch": 1,
  "noAck": false,
  "autoCreateQueue": true,
  "durable": true
}
```

**Output (on message received):**
```json
{
  "queue": "orders",
  "message": { "orderId": "4521", "status": "created", "amount": 149.99 },
  "properties": { "deliveryMode": 2, "contentType": "application/json" },
  "fields": { "deliveryTag": 1, "routingKey": "orders", "redelivered": false },
  "timestamp": 1741600000123
}
```

---

### Example: High-Throughput Consumer

Process up to 5 messages in parallel.

**Configuration:**
```json
{
  "host": "rabbitmq.example.com",
  "port": 5672,
  "username": "fusion",
  "password": "secret",
  "queue": "notifications",
  "prefetch": 5,
  "noAck": true,
  "autoCreateQueue": false
}
```

> **Note:** With `noAck: true`, messages are acknowledged immediately upon delivery and will not be redelivered on failure. Use only for non-critical, idempotent operations.

---

### Example: Consumer with Specific vhost

Connect to a non-default virtual host.

**Configuration:**
```json
{
  "host": "rabbitmq.example.com",
  "port": 5672,
  "username": "fusion",
  "password": "secret",
  "vhost": "production",
  "queue": "audit-log",
  "prefetch": 1,
  "noAck": false,
  "durable": true
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Sample Workflow: Order Processing Pipeline

Consume orders from RabbitMQ, validate, and save to PostgreSQL.

```json
{
  "nodes": [
    {
      "id": "mq-consumer",
      "type": "rabbitmq-trigger",
      "position": { "x": 100, "y": 100 },
      "config": {
        "host": "rabbitmq.example.com",
        "port": 5672,
        "username": "fusion",
        "password": "secret",
        "queue": "orders",
        "prefetch": 1,
        "noAck": false
      }
    },
    {
      "id": "validate",
      "type": "function",
      "position": { "x": 300, "y": 100 },
      "config": {
        "code": "const order = input.message; if (!order.orderId) throw new Error('Missing orderId'); return order;"
      }
    },
    {
      "id": "save-order",
      "type": "postgres-action",
      "position": { "x": 500, "y": 100 },
      "config": {
        "host": "db.example.com",
        "database": "myapp",
        "tableName": "orders",
        "operation": "Insert",
        "insertParams": {
          "rows": [
            { "rowName": "order_id", "rowValue": "{{input.orderId}}" },
            { "rowName": "customer_id", "rowValue": "{{input.customerId}}" },
            { "rowName": "amount", "rowValue": "{{input.amount}}" },
            { "rowName": "status", "rowValue": "{{input.status}}" }
          ]
        }
      }
    }
  ]
}
```

### Common Patterns

- **Reliable Worker:** RabbitMQ Trigger (noAck: false) → Process → Ack on success / Nack on error
- **Rate-Limited Pipeline:** RabbitMQ Trigger (prefetch: 1) → Slow external API call → Continue
- **Fan-In:** Multiple producers → Single RabbitMQ Trigger → Unified processing workflow
- **Dead Letter Queue:** Error output → [RabbitMQ Action](./rabbitmq-action.md) (publish to DLQ)
- **Chained Workflows:** [RabbitMQ Action](./rabbitmq-action.md) publish → RabbitMQ Trigger consume

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Failed to initialize RabbitMQ`

**Cause:** Broker unreachable, wrong credentials, or vhost does not exist.

**Solution:** Verify `host`, `port`, `username`, `password`, and `vhost`. Confirm the user has permission on the vhost with `rabbitmqctl list_permissions -p <vhost>`.

#### Node starts but never triggers

**Cause:** No messages are being published to the subscribed queue.

**Solution:** Use the [RabbitMQ Action](./rabbitmq-action.md) node with `publish` to send a test message. Use `getQueueInfo` to confirm the queue has messages and that the consumer is connected.

#### Messages pile up in the queue

**Cause:** `prefetch` is set to `1` and the workflow takes too long to process each message.

**Solution:** Increase `prefetch` if the workflow is safe to run concurrently. Or optimize the downstream nodes to reduce processing time.

#### Message delivered with `redelivered: true`

**Cause:** A previous consumer received the message but did not acknowledge it (e.g. the node crashed mid-processing).

**Solution:** This is expected behavior with `noAck: false`. Add idempotency logic in a Function node (e.g. check if the order ID already exists before inserting) to safely handle redelivered messages.

#### `message` is a raw string instead of an object

**Cause:** The producer sent a non-JSON payload.

**Solution:** Use a Function node after the trigger to parse the format manually (e.g. CSV, XML, plain text).

### Error Codes

| Code | Message | Solution |
|------|---------|----------|
| `ECONNREFUSED` | Connection refused | Check broker host, port, and firewall |
| `ACCESS_REFUSED` | Authentication failed | Verify credentials and vhost permissions |
| `NOT_FOUND` | Queue does not exist | Enable `autoCreateQueue: true` or create the queue manually |
| `PRECONDITION_FAILED` | Queue redeclaration conflict | The existing queue has different parameters — match them or delete and recreate |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [RabbitMQ Action](./rabbitmq-action.md) - Publish messages and manage queues/exchanges
- [Kafka Trigger](./kafka-trigger.md) - Consumer trigger for Apache Kafka
- [Redis Subscribe](./redis-subscribe.md) - Lightweight Pub/Sub trigger

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-03-10 | Initial release |

<!-- /SECTION: changelog -->
