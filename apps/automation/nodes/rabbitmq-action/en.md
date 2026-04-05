---
node_id: "rabbitmq-action"
title: "RabbitMQ Action"
description: "Perform RabbitMQ operations — publish messages, manage queues, bind/unbind, and manage exchanges"
category: "messaging"
subcategory: "rabbitmq"
version: "1.0.0"
language: "en"
last_updated: "2026-03-10"
author: "Fusion Team"
tags:
  - rabbitmq
  - messaging
  - amqp
  - queue
  - exchange
  - action
related_nodes:
  - rabbitmq-trigger
  - kafka-publish
  - redis-action
---

<!-- SECTION: header -->
# RabbitMQ Action

> **Category:** Messaging | **Type:** Action Node

Connect to a RabbitMQ broker and perform queue or exchange operations — publish messages, create/delete/purge queues, bind/unbind queues to exchanges, and manage exchanges.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **RabbitMQ Action** node connects to a RabbitMQ broker using the AMQP protocol and executes one of nine available operations. It covers the full lifecycle of queue and exchange management, making it suitable for both simple message publishing and advanced AMQP topology setup within automation workflows.

### Key Features

- **Nine Operations:** publish, createQueue, getQueueInfo, purgeQueue, deleteQueue, bindQueue, unbindQueue, createExchange, deleteExchange
- **Flexible Publishing:** Publish to a queue directly or route via an exchange with a routing key
- **Exchange Types:** Supports `direct`, `fanout`, `topic`, and `headers` exchanges
- **Durability Control:** Configurable `durable` flag for queues and exchanges
- **Persistent Messages:** Optional `persistent` flag ensures messages survive broker restarts
- **vhost Support:** Target any virtual host on the broker
- **Expression Support:** `queue`, `message`, `exchange`, and `routingKey` accept dynamic expressions

### Use Cases

- Publish domain events to RabbitMQ from a workflow action
- Set up AMQP topology (queues, exchanges, bindings) as part of an infrastructure workflow
- Inspect queue state (message count, consumer count) for monitoring
- Purge test queues between workflow runs
- Route messages to specific consumers via exchange bindings

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

### Common Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `enum` | ✅ Yes | — | Operation to perform (see below) |
| `durable` | `boolean` | ❌ No | `true` | Make queues/exchanges survive broker restarts |
| `persistent` | `boolean` | ❌ No | `true` | Make published messages survive broker restarts |

### Available Operations

| Operation | Description |
|-----------|-------------|
| `publish` | Publish a message to a queue or exchange |
| `createQueue` | Declare a new queue |
| `getQueueInfo` | Get message and consumer count for a queue |
| `purgeQueue` | Remove all messages from a queue |
| `deleteQueue` | Delete a queue |
| `bindQueue` | Bind a queue to an exchange with a routing key |
| `unbindQueue` | Remove a binding between a queue and an exchange |
| `createExchange` | Declare a new exchange |
| `deleteExchange` | Delete an exchange |

---

### Operation: publish

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `queue` | `string` | Conditional | Target queue name — used as routing key when no exchange is specified |
| `exchange` | `string` | ❌ No | Exchange to publish to — leave empty to publish directly to the default exchange |
| `routingKey` | `string` | ❌ No | Routing key for the message — defaults to `queue` if not set |
| `message` | `string` | ✅ Yes | Message payload (string or JSON string) |
| `persistent` | `boolean` | ❌ No | Persist message to disk (`deliveryMode: 2`) |

### Operation: createQueue

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `queue` | `string` | ✅ Yes | Name of the queue to create |
| `durable` | `boolean` | ❌ No | Survive broker restarts (default: `true`) |

### Operation: getQueueInfo

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `queue` | `string` | ✅ Yes | Name of the queue to inspect |

### Operation: purgeQueue / deleteQueue

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `queue` | `string` | ✅ Yes | Name of the queue to purge or delete |

### Operation: bindQueue / unbindQueue

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `queue` | `string` | ✅ Yes | Queue to bind or unbind |
| `exchange` | `string` | ✅ Yes | Exchange to bind/unbind from |
| `routingKey` | `string` | ✅ Yes | Routing key for the binding |

### Operation: createExchange

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `exchange` | `string` | ✅ Yes | Name of the exchange to create |
| `exchangeType` | `enum` | ✅ Yes | Exchange type: `direct`, `fanout`, `topic`, or `headers` |
| `durable` | `boolean` | ❌ No | Survive broker restarts (default: `true`) |

### Operation: deleteExchange

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `exchange` | `string` | ✅ Yes | Name of the exchange to delete |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Incoming workflow data — available via expressions in `queue`, `message`, `exchange`, `routingKey` |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `output` | `object` | Result of the executed operation |
| `error` | `Error` | Emitted if the operation or connection fails |

### Output Schemas by Operation

**`publish`**
```json
{
  "success": true,
  "queue": "orders",
  "message": "{\"orderId\": \"4521\", \"status\": \"created\"}"
}
```

**`createQueue`**
```json
{
  "success": true,
  "queue": "orders",
  "durable": true
}
```

**`getQueueInfo`**
```json
{
  "queue": "orders",
  "messageCount": 42,
  "consumerCount": 3
}
```

**`purgeQueue`**
```json
{
  "success": true,
  "queue": "orders",
  "messagesPurged": 42
}
```

**`deleteQueue`**
```json
{
  "success": true,
  "queue": "orders",
  "messageCount": 0
}
```

**`bindQueue` / `unbindQueue`**
```json
{
  "success": true,
  "queue": "orders",
  "exchange": "events",
  "routingKey": "order.created"
}
```

**`createExchange`**
```json
{
  "success": true,
  "exchange": "events",
  "type": "topic",
  "durable": true
}
```

**`deleteExchange`**
```json
{
  "success": true,
  "exchange": "events"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example: Publish Directly to a Queue

Send an order event directly to the `orders` queue.

**Configuration:**
```json
{
  "host": "rabbitmq.example.com",
  "port": 5672,
  "username": "fusion",
  "password": "secret",
  "vhost": "/",
  "operation": "publish",
  "queue": "orders",
  "message": "{{JSON.stringify(input.order)}}",
  "persistent": true
}
```

**Output:**
```json
{
  "success": true,
  "queue": "orders",
  "message": "{\"orderId\": \"4521\", \"status\": \"created\"}"
}
```

---

### Example: Publish via Exchange with Routing Key

Route a message through a `topic` exchange.

**Configuration:**
```json
{
  "host": "rabbitmq.example.com",
  "port": 5672,
  "username": "fusion",
  "password": "secret",
  "operation": "publish",
  "exchange": "events",
  "routingKey": "order.created",
  "message": "{{JSON.stringify(input)}}",
  "persistent": true
}
```

---

### Example: Inspect Queue State

Check how many messages and consumers are on the `orders` queue.

**Configuration:**
```json
{
  "host": "rabbitmq.example.com",
  "port": 5672,
  "username": "fusion",
  "password": "secret",
  "operation": "getQueueInfo",
  "queue": "orders"
}
```

**Output:**
```json
{
  "queue": "orders",
  "messageCount": 17,
  "consumerCount": 2
}
```

---

### Example: Create an Exchange and Bind a Queue

Set up a topic exchange and bind a queue to it.

**Step 1 — Create the exchange:**
```json
{
  "operation": "createExchange",
  "exchange": "events",
  "exchangeType": "topic",
  "durable": true
}
```

**Step 2 — Create the queue:**
```json
{
  "operation": "createQueue",
  "queue": "order-processor",
  "durable": true
}
```

**Step 3 — Bind the queue:**
```json
{
  "operation": "bindQueue",
  "queue": "order-processor",
  "exchange": "events",
  "routingKey": "order.*"
}
```

---

### Example: Purge a Queue

Clear all pending messages from a queue.

**Configuration:**
```json
{
  "host": "rabbitmq.example.com",
  "port": 5672,
  "username": "fusion",
  "password": "secret",
  "operation": "purgeQueue",
  "queue": "staging-events"
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Sample Workflow: Webhook to RabbitMQ

Receive an HTTP webhook and forward the payload to a RabbitMQ queue for async processing.

```json
{
  "nodes": [
    {
      "id": "receive-webhook",
      "type": "webhook-trigger",
      "position": { "x": 100, "y": 100 }
    },
    {
      "id": "serialize",
      "type": "function",
      "position": { "x": 300, "y": 100 },
      "config": {
        "code": "return { ...input, receivedAt: new Date().toISOString() };"
      }
    },
    {
      "id": "publish-to-mq",
      "type": "rabbitmq-action",
      "position": { "x": 500, "y": 100 },
      "config": {
        "host": "rabbitmq.example.com",
        "port": 5672,
        "username": "fusion",
        "password": "secret",
        "operation": "publish",
        "exchange": "events",
        "routingKey": "{{input.eventType}}",
        "message": "{{JSON.stringify(input)}}",
        "persistent": true
      }
    }
  ]
}
```

### Common Patterns

- **Async Processing:** Action completes → RabbitMQ Publish → [RabbitMQ Trigger](./rabbitmq-trigger.md) → Worker workflow
- **Topology Setup:** Cron or manual trigger → createExchange → createQueue → bindQueue
- **Queue Monitoring:** Cron → getQueueInfo → Check `messageCount` → Alert if above threshold
- **Cleanup Job:** Cron → purgeQueue (staging) → Log result
- **Fan-Out:** Publish to `fanout` exchange → All bound queues receive the message

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Failed to initialize RabbitMQ`

**Cause:** Broker is unreachable, credentials are wrong, or the vhost does not exist.

**Solution:** Verify `host`, `port`, `username`, `password`, and `vhost`. Use `rabbitmqctl list_vhosts` to confirm the vhost exists. Check network access and firewall rules.

#### `Message is required for publish`

**Cause:** The `message` field was left empty.

**Solution:** Provide a non-empty message value. If using a dynamic expression, verify that it resolves to a non-empty string.

#### `Queue, exchange, and routing key are required` on bindQueue

**Cause:** One of `queue`, `exchange`, or `routingKey` is missing.

**Solution:** All three fields are required for `bindQueue` and `unbindQueue`. Verify each field is filled in.

#### `Exchange name and type are required` on createExchange

**Cause:** The `exchange` or `exchangeType` field is missing.

**Solution:** Provide a valid exchange name and select one of the supported types: `direct`, `fanout`, `topic`, or `headers`.

#### Published message not received by consumer

**Cause:** The queue binding or routing key may not match the consumer's subscription.

**Solution:** Use `getQueueInfo` to verify the queue exists and has consumers. Check that the `routingKey` matches the binding on the exchange.

### Error Codes

| Code | Message | Solution |
|------|---------|----------|
| `ECONNREFUSED` | Connection refused | Check broker host, port, and firewall |
| `ACCESS_REFUSED` | Access denied | Verify credentials and vhost permissions |
| `NOT_FOUND` | Queue or exchange not found | Create the resource before referencing it |
| `PRECONDITION_FAILED` | Conflicting queue/exchange declaration | Existing declaration has different parameters |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [RabbitMQ Trigger](./rabbitmq-trigger.md) - Consume messages from a RabbitMQ queue
- [Kafka Publish](./kafka-publish.md) - Alternative high-throughput message publishing
- [Redis Action](./redis-action.md) - Lightweight Pub/Sub alternative

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-03-10 | Initial release |

<!-- /SECTION: changelog -->
