---
node_id: "kafka-publish"
title: "Kafka Publish"
description: "Publish messages to Apache Kafka topics with support for SSL, SASL authentication, message keys, headers, and partition targeting"
category: "messaging"
subcategory: "kafka"
version: "1.0.0"
language: "en"
last_updated: "2026-03-10"
author: "Fusion Team"
tags:
  - kafka
  - messaging
  - pubsub
  - streaming
  - action
related_nodes:
  - kafka-trigger
  - redis-action
  - function
---

<!-- SECTION: header -->
# Kafka Publish

> **Category:** Messaging | **Type:** Action Node

Publish a message to an Apache Kafka topic. Supports SSL encryption, SASL authentication, message keys, custom headers, and partition targeting.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Kafka Publish** node connects to a Kafka broker as a producer and sends a single message to a target topic. It supports all production-grade security features (SSL, SASL PLAIN, SCRAM-SHA-256, SCRAM-SHA-512) and provides fine-grained control over delivery acknowledgment via the `acks` setting.

### Key Features

- **Topic Publishing:** Send a message to any Kafka topic
- **Message Key:** Route messages to a consistent partition using a key
- **Custom Headers:** Attach metadata headers (JSON object) to each message
- **Partition Targeting:** Publish to a specific partition directly
- **SSL Support:** Encrypt broker connections with TLS
- **SASL Authentication:** Supports `plain`, `scram-sha-256`, and `scram-sha-512`
- **Acknowledgment Control:** Configure delivery guarantee with `acks` (`0`, `1`, `-1`)
- **Expression Support:** `topic`, `message`, `messageKey`, `headers`, and `partition` accept dynamic expressions

### Use Cases

- Emit domain events (order created, user registered) to a Kafka topic
- Feed processed data from a workflow into a Kafka-based data pipeline
- Trigger downstream consumers after completing a workflow action
- Forward webhook payloads to a Kafka topic for async processing
- Publish enriched or transformed records to a topic for analytics

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Connection Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `brokers` | `string` | ✅ Yes | — | Comma-separated list of broker addresses (e.g. `broker1:9092,broker2:9092`) |
| `clientId` | `string` | ❌ No | `fusion-kafka-client` | Kafka client identifier |
| `ssl` | `boolean` | ❌ No | `false` | Enable TLS/SSL encryption for the broker connection |

### SASL Authentication (optional)

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `saslEnabled` | `boolean` | ❌ No | `false` | Enable SASL authentication |
| `saslMechanism` | `enum` | Conditional | — | Authentication mechanism — `plain`, `scram-sha-256`, `scram-sha-512` |
| `saslUsername` | `string` | Conditional | — | SASL username |
| `saslPassword` | `string` | Conditional | — | SASL password |

> `saslMechanism`, `saslUsername`, and `saslPassword` are required when `saslEnabled` is `true`.

### Message Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `topic` | `string` | ✅ Yes | — | Kafka topic to publish to (supports expressions) |
| `message` | `string` | ✅ Yes | — | Message payload — string or JSON string (supports expressions) |
| `messageKey` | `string` | ❌ No | — | Partition routing key (supports expressions) |
| `headers` | `string` | ❌ No | — | JSON object of string headers, e.g. `{"source": "fusion"}` (supports expressions) |
| `partition` | `number` | ❌ No | — | Target partition index — if omitted, Kafka uses the default partitioner |

### Delivery Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `acks` | `enum` | ❌ No | `-1` | Acknowledgment mode: `0` = no ack, `1` = leader ack, `-1` = all replicas ack |

### `acks` Reference

| Value | Mode | Description |
|-------|------|-------------|
| `-1` | **All** | Wait for all in-sync replicas to acknowledge — strongest durability guarantee |
| `1` | **Leader** | Wait for the partition leader to acknowledge |
| `0` | **No ack** | Fire and forget — highest throughput, no delivery guarantee |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Incoming workflow data — available via expressions in `topic`, `message`, `messageKey`, etc. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `output` | `object` | Publish result |
| `error` | `Error` | Emitted if the message fails to be published |

### Output Schema

```json
{
  "success": true,
  "topic": "orders",
  "messageKey": "order-4521",
  "result": [
    {
      "topicName": "orders",
      "partition": 0,
      "errorCode": 0,
      "offset": "142",
      "timestamp": "1741600000000",
      "logStartOffset": "0"
    }
  ]
}
```

| Field | Type | Description |
|-------|------|-------------|
| `success` | `boolean` | Always `true` if the node did not throw |
| `topic` | `string` | The topic the message was published to |
| `messageKey` | `string \| null` | The key used for partition routing, or `null` if not set |
| `result` | `array` | Raw KafkaJS `RecordMetadata` — includes partition, offset, and timestamp |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example: Basic Publish

Send a simple JSON event to a Kafka topic.

**Configuration:**
```json
{
  "brokers": "kafka.example.com:9092",
  "topic": "user-events",
  "message": "{\"event\": \"user_registered\", \"userId\": \"{{input.userId}}\"}",
  "acks": "-1"
}
```

**Output:**
```json
{
  "success": true,
  "topic": "user-events",
  "messageKey": null,
  "result": [{ "topicName": "user-events", "partition": 2, "offset": "87", "errorCode": 0 }]
}
```

---

### Example: Publish with Key and Headers

Route an order event to a consistent partition using the order ID as the key.

**Configuration:**
```json
{
  "brokers": "kafka.example.com:9092",
  "topic": "orders",
  "message": "{{JSON.stringify(input.order)}}",
  "messageKey": "order-{{input.order.id}}",
  "headers": "{\"source\": \"fusion-workflow\", \"env\": \"production\"}",
  "acks": "1"
}
```

---

### Example: SSL + SASL Authentication

Connect to a secured Kafka cluster using SCRAM-SHA-256.

**Configuration:**
```json
{
  "brokers": "kafka-secure.example.com:9093",
  "clientId": "fusion-producer",
  "ssl": true,
  "saslEnabled": true,
  "saslMechanism": "scram-sha-256",
  "saslUsername": "fusion-user",
  "saslPassword": "secret",
  "topic": "secure-events",
  "message": "{{JSON.stringify(input)}}",
  "acks": "-1"
}
```

---

### Example: Target a Specific Partition

Force the message into partition `0` (e.g. for ordered processing).

**Configuration:**
```json
{
  "brokers": "localhost:9092",
  "topic": "ordered-events",
  "message": "{{JSON.stringify(input)}}",
  "partition": 0,
  "acks": "-1"
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Sample Workflow: Webhook to Kafka

Receive a webhook and forward the payload to Kafka for downstream processing.

```json
{
  "nodes": [
    {
      "id": "webhook",
      "type": "webhook-trigger",
      "position": { "x": 100, "y": 100 }
    },
    {
      "id": "enrich",
      "type": "function",
      "position": { "x": 300, "y": 100 },
      "config": {
        "code": "return { ...input, receivedAt: new Date().toISOString(), source: 'fusion' };"
      }
    },
    {
      "id": "publish-to-kafka",
      "type": "kafka-publish",
      "position": { "x": 500, "y": 100 },
      "config": {
        "brokers": "kafka.example.com:9092",
        "topic": "{{input.eventType}}",
        "message": "{{JSON.stringify(input)}}",
        "messageKey": "{{input.id}}",
        "headers": "{\"source\": \"fusion\"}",
        "acks": "-1"
      }
    }
  ]
}
```

### Common Patterns

- **Event Sourcing:** Action completes → Kafka Publish (domain event)
- **Data Pipeline Entry:** Webhook → Kafka Publish → Downstream consumers
- **Fan-Out:** One workflow → Kafka Publish → Multiple consumer groups
- **Async Processing:** Kafka Publish → [Kafka Trigger](./kafka-trigger.md) → Worker workflow
- **Dead Letter Queue:** Error handler → Kafka Publish to DLQ topic

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Failed to initialize Kafka producer`

**Cause:** The broker address is unreachable, SSL is misconfigured, or SASL credentials are invalid.

**Solution:** Verify the `brokers` value (format: `host:port`). Check network access and firewall rules. If using SSL, ensure the broker's certificate is valid. If using SASL, confirm credentials and the correct mechanism.

#### `Topic is required`

**Cause:** The `topic` field was left empty.

**Solution:** Provide a non-empty topic name. If using a dynamic expression, verify that it resolves to a valid string.

#### Message published but no consumer receives it

**Cause:** The consumer group may not be subscribed to the topic, or there is a topic/partition mismatch.

**Solution:** Confirm that the [Kafka Trigger](./kafka-trigger.md) node (or your consumer) is subscribed to the exact same topic name. Check partition assignment if targeting a specific partition.

#### `acks: 0` — messages appear lost

**Cause:** With `acks: 0`, the producer does not wait for any acknowledgment. Messages can be lost if the broker is under load.

**Solution:** Use `acks: 1` or `acks: -1` for stronger delivery guarantees in production.

### Error Codes

| Code | Message | Solution |
|------|---------|----------|
| `ECONNREFUSED` | Broker connection refused | Check broker host, port, and firewall |
| `SASL_AUTHENTICATION_FAILED` | Invalid credentials | Verify `saslUsername` and `saslPassword` |
| `LEADER_NOT_AVAILABLE` | Topic not found or not ready | Confirm topic exists on the broker |
| `REQUEST_TIMED_OUT` | Broker did not respond in time | Check broker health and network latency |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Kafka Trigger](./kafka-trigger.md) - Trigger workflows from incoming Kafka messages
- [Redis Action](./redis-action.md) - Lightweight Pub/Sub alternative
- [Function](./function.md) - Transform or serialize data before publishing

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-03-10 | Initial release |

<!-- /SECTION: changelog -->
