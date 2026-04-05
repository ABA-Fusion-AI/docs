---
node_id: "kafka-trigger"
title: "Kafka Trigger"
description: "Trigger workflow execution when messages arrive in Apache Kafka topics — supports consumer groups, SSL, SASL, and pause/resume"
category: "triggers"
subcategory: "kafka"
version: "1.0.0"
language: "en"
last_updated: "2026-03-10"
author: "Fusion Team"
tags:
  - kafka
  - trigger
  - messaging
  - streaming
  - consumer
  - real-time
related_nodes:
  - kafka-publish
  - redis-subscribe
  - function
---

<!-- SECTION: header -->
# Kafka Trigger

> **Category:** Triggers | **Type:** Trigger Node

Subscribe to one or more Apache Kafka topics as a consumer and trigger a workflow execution for every incoming message. Supports consumer groups, SSL, SASL, offset management, and pause/resume control.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Kafka Trigger** node acts as a Kafka consumer within a named consumer group. It subscribes to one or more topics and emits a workflow trigger for each message received, passing the full message payload including topic, partition, offset, key, value, headers, and timestamp.

The node supports all production-grade security features and provides reliable consumption with configurable auto-commit and heartbeat intervals. It can also be paused and resumed without losing its consumer group offset.

### Key Features

- **Multi-Topic:** Subscribe to multiple Kafka topics in a single node
- **Consumer Group:** Participate in a named consumer group for distributed consumption
- **Offset Control:** Choose to start from the beginning (`fromBeginning`) or from the latest offset
- **Auto-Commit:** Automatically commit offsets at a configurable interval
- **Pause / Resume:** Temporarily suspend consumption without disconnecting
- **SSL + SASL:** Supports TLS, PLAIN, SCRAM-SHA-256, and SCRAM-SHA-512
- **JSON Auto-Parsing:** Message values are automatically parsed as JSON when possible
- **Rich Output:** Each trigger includes topic, partition, offset, key, value, headers, and timestamp

### Use Cases

- Process orders or events produced by backend services
- Build real-time data pipelines fed by Kafka streams
- Chain workflows using [Kafka Publish](./kafka-publish.md) as the producer
- React to IoT or telemetry data arriving on Kafka topics
- Implement event-driven microservice orchestration

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Connection Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `brokers` | `string` | ✅ Yes | — | Comma-separated broker addresses (e.g. `broker1:9092,broker2:9092`) |
| `clientId` | `string` | ❌ No | `fusion-kafka-client` | Kafka client identifier |
| `ssl` | `boolean` | ❌ No | `false` | Enable TLS/SSL encryption for broker connections |

### SASL Authentication (optional)

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `saslEnabled` | `boolean` | ❌ No | `false` | Enable SASL authentication |
| `saslMechanism` | `enum` | Conditional | — | `plain`, `scram-sha-256`, or `scram-sha-512` |
| `saslUsername` | `string` | Conditional | — | SASL username |
| `saslPassword` | `string` | Conditional | — | SASL password |

> `saslMechanism`, `saslUsername`, and `saslPassword` are required when `saslEnabled` is `true`.

### Consumer Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `groupId` | `string` | ✅ Yes | — | Consumer group ID — all nodes with the same ID share partitions |
| `topics` | `string[]` | ✅ Yes | — | List of topic names to subscribe to |
| `fromBeginning` | `boolean` | ❌ No | `false` | Start consuming from the earliest available offset |

### Session & Commit Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `autoCommit` | `boolean` | ❌ No | `true` | Automatically commit offsets after processing |
| `autoCommitInterval` | `number` | ❌ No | `5000` | Auto-commit interval in milliseconds |
| `sessionTimeout` | `number` | ❌ No | `30000` | Max time (ms) without a heartbeat before the broker considers the consumer dead |
| `heartbeatInterval` | `number` | ❌ No | `3000` | Interval (ms) between heartbeats sent to the broker |

> **Best practice:** `heartbeatInterval` should be at most one-third of `sessionTimeout`.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

This is a trigger node — it activates automatically when a message arrives on a subscribed topic. It does not accept inputs from other nodes.

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted for each received message |
| `error` | `Error` | Emitted on processing or connection errors |

### Output Schema (`success`)

```json
{
  "topic": "orders",
  "partition": 0,
  "offset": "142",
  "key": "order-4521",
  "value": {
    "orderId": "4521",
    "customerId": "usr-99",
    "amount": 149.99,
    "status": "created"
  },
  "headers": {
    "source": "fusion-workflow",
    "env": "production"
  },
  "timestamp": "1741600000000",
  "receivedAt": 1741600000123
}
```

| Field | Type | Description |
|-------|------|-------------|
| `topic` | `string` | Topic the message was consumed from |
| `partition` | `number` | Partition index |
| `offset` | `string` | Message offset within the partition |
| `key` | `string \| null` | Message routing key, or `null` if not set |
| `value` | `any` | Message payload — auto-parsed as JSON if valid, otherwise raw string |
| `headers` | `object` | Key-value string headers attached to the message |
| `timestamp` | `string` | Kafka broker timestamp (Unix ms as string) |
| `receivedAt` | `number` | Timestamp when the node received the message (`Date.now()`) |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example: Basic Consumer

Trigger a workflow for every message on the `orders` topic.

**Configuration:**
```json
{
  "brokers": "kafka.example.com:9092",
  "groupId": "fusion-order-processor",
  "topics": ["orders"],
  "fromBeginning": false,
  "autoCommit": true,
  "autoCommitInterval": 5000
}
```

**Output (on message received):**
```json
{
  "topic": "orders",
  "partition": 1,
  "offset": "88",
  "key": "order-100",
  "value": { "orderId": "100", "status": "created", "amount": 59.90 },
  "headers": {},
  "timestamp": "1741600000000",
  "receivedAt": 1741600000050
}
```

---

### Example: Multi-Topic Consumer

Listen to both `orders` and `refunds` topics in the same node.

**Configuration:**
```json
{
  "brokers": "kafka.example.com:9092",
  "groupId": "fusion-finance-processor",
  "topics": ["orders", "refunds"],
  "fromBeginning": false
}
```

> **Tip:** Use a Function node after this trigger to branch on `input.topic`:
> ```js
> if (input.topic === 'orders') { ... }
> if (input.topic === 'refunds') { ... }
> ```

---

### Example: Replay from Beginning

Re-process all historical messages from the start of the topic.

**Configuration:**
```json
{
  "brokers": "kafka.example.com:9092",
  "groupId": "fusion-replay-job",
  "topics": ["user-events"],
  "fromBeginning": true,
  "autoCommit": true
}
```

> **Note:** Use a unique `groupId` for replay jobs to avoid interfering with production consumers.

---

### Example: Secured Cluster with SASL

Connect to a production Kafka cluster with SSL and SCRAM-SHA-512.

**Configuration:**
```json
{
  "brokers": "kafka-secure.example.com:9093",
  "clientId": "fusion-consumer",
  "ssl": true,
  "saslEnabled": true,
  "saslMechanism": "scram-sha-512",
  "saslUsername": "fusion-user",
  "saslPassword": "secret",
  "groupId": "fusion-secure-group",
  "topics": ["secure-events"],
  "sessionTimeout": 30000,
  "heartbeatInterval": 3000
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Sample Workflow: Kafka → Database Pipeline

Consume messages from a Kafka topic and insert each one into PostgreSQL.

```json
{
  "nodes": [
    {
      "id": "kafka-consumer",
      "type": "kafka-trigger",
      "position": { "x": 100, "y": 100 },
      "config": {
        "brokers": "kafka.example.com:9092",
        "groupId": "fusion-db-writer",
        "topics": ["user-events"],
        "fromBeginning": false
      }
    },
    {
      "id": "validate",
      "type": "function",
      "position": { "x": 300, "y": 100 },
      "config": {
        "code": "if (!input.value?.userId) throw new Error('Missing userId'); return input.value;"
      }
    },
    {
      "id": "save-to-db",
      "type": "postgres-action",
      "position": { "x": 500, "y": 100 },
      "config": {
        "host": "db.example.com",
        "port": 5432,
        "user": "app_user",
        "password": "secret",
        "database": "myapp",
        "tableName": "user_events",
        "operation": "Insert",
        "insertParams": {
          "rows": [
            { "rowName": "user_id", "rowValue": "{{input.userId}}" },
            { "rowName": "event", "rowValue": "{{input.event}}" },
            { "rowName": "created_at", "rowValue": "{{new Date().toISOString()}}" }
          ]
        }
      }
    }
  ]
}
```

### Common Patterns

- **Event Processing:** Kafka Trigger → Function (validate/transform) → Action node
- **Fan-Out Consumer:** Kafka Trigger → Multiple parallel branches
- **Dead Letter Queue:** Kafka Trigger error output → [Kafka Publish](./kafka-publish.md) to DLQ topic
- **Chained Workflows:** [Kafka Publish](./kafka-publish.md) → Kafka Trigger (downstream workflow)
- **Real-Time Dashboard:** Kafka Trigger → Aggregate → Push to frontend via SignalR

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Failed to initialize Kafka consumer`

**Cause:** Broker is unreachable, SSL/SASL is misconfigured, or `groupId` is empty.

**Solution:** Verify `brokers` format (`host:port`), network access, and security settings. Ensure `groupId` is a non-empty string.

#### Node starts but never triggers

**Cause:** No messages are being produced to the subscribed topics, or the consumer group has already committed offsets to the latest position.

**Solution:** Verify that messages are being published to the topic (use a Kafka CLI tool or the [Kafka Publish](./kafka-publish.md) node). Use `fromBeginning: true` with a new `groupId` to replay historical messages.

#### `allowAutoTopicCreation: false` — topic not found

**Cause:** The specified topic does not exist on the broker, and auto-creation is disabled.

**Solution:** Create the topic on the Kafka broker before starting the node. Auto-topic creation is intentionally disabled in this node to avoid silent misconfiguration.

#### Consumer group rebalances frequently

**Cause:** `sessionTimeout` is too low or `heartbeatInterval` is too high relative to `sessionTimeout`.

**Solution:** Ensure `heartbeatInterval` is at most one-third of `sessionTimeout`. Increase `sessionTimeout` if processing per message takes a long time.

#### `value` is a raw string instead of a parsed object

**Cause:** The message value was not valid JSON.

**Solution:** This is expected behavior — the node returns the raw string when JSON parsing fails. Use a Function node to handle non-JSON formats (CSV, XML, plain text, etc.).

### Error Codes

| Code | Message | Solution |
|------|---------|----------|
| `ECONNREFUSED` | Broker connection refused | Check broker host, port, and firewall |
| `SASL_AUTHENTICATION_FAILED` | Invalid credentials | Verify `saslUsername` and `saslPassword` |
| `GROUP_COORDINATOR_NOT_AVAILABLE` | Consumer group error | Retry — broker may be electing a coordinator |
| `SESSION_EXPIRED` | Heartbeat missed | Increase `sessionTimeout` or reduce processing time per message |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Kafka Publish](./kafka-publish.md) - Publish messages to Kafka topics
- [Redis Subscribe](./redis-subscribe.md) - Lightweight alternative for Pub/Sub triggers
- [Function](./function.md) - Parse and transform incoming Kafka message payloads

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-03-10 | Initial release |

<!-- /SECTION: changelog -->
