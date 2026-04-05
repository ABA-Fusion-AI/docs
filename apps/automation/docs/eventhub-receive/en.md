---
node_id: "eventhub-receive"
title: "Event Hub Receive"
description: "Trigger workflows when events arrive in an Azure Event Hub — real-time consumer with configurable starting position, batch size, and consumer group"
category: "integrations"
subcategory: "messaging"
version: "1.0.0"
language: "en"
last_updated: "2026-03-10"
author: "Fusion Team"
tags:
  - azure
  - event-hubs
  - messaging
  - streaming
  - consumer
  - trigger
related_nodes:
  - eventhub-send
  - eventhub-admin
  - kafka-trigger
  - rabbitmq-trigger
---

<!-- SECTION: header -->
# Event Hub Receive

> **Category:** Integrations | **Type:** Trigger Node

Subscribe to an Azure Event Hub and trigger a workflow for each received event. Supports configurable consumer groups, starting positions, batch sizes, and partition-level session tracking.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Event Hub Receive** node uses the `@azure/event-hubs` `EventHubConsumerClient` to subscribe to all partitions of an Event Hub. It processes events in batches and emits one workflow trigger per event.

The node supports two starting positions — `latest` (only new events after the workflow starts) and `earliest` (replay all events from the beginning of the retention window). Each emitted payload includes full event metadata: body, properties, system properties, partition information, offset, and sequence number.

### Key Features

- **All-Partition Consumption:** Automatically subscribes to all available partitions
- **Configurable Starting Position:** `latest` for live processing, `earliest` for replay/backfill
- **Batch Processing:** Configurable `maxBatchSize` and `maxWaitTimeInSeconds` for throughput tuning
- **Consumer Groups:** Supports custom consumer groups for independent parallel consumers
- **Partition Context:** Each emitted event includes its `partitionId`, `offset`, and `sequenceNumber`
- **Pause / Resume:** Closes and recreates the subscription without tearing down the client

### Use Cases

- Process incoming IoT telemetry events from field devices in real time
- Replay historical events from an Event Hub to backfill a database or reprocess failed records
- Consume events published by multiple upstream services into a single Event Hub for unified processing
- Bridge Azure Event Hubs data into other systems (Slack notifications, Jira issues, database inserts)

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `connectionString` | `string` | ✅ Yes | — | Azure Event Hub connection string (SAS format) |
| `eventHubName` | `string` | ✅ Yes | — | Name of the Event Hub to consume from |
| `consumerGroup` | `string` | ✅ Yes | `$Default` | Consumer group name — use `$Default` unless you have multiple independent consumers |
| `startingPosition` | `enum` | ❌ No | `latest` | Where to start reading: `latest` (only new events) or `earliest` (replay from start) |
| `maxBatchSize` | `number` | ❌ No | `50` | Maximum number of events to receive per batch (minimum 1) |
| `maxWaitTimeInSeconds` | `number` | ❌ No | `5` | How long to wait for a batch to fill before processing (minimum 1) |

### Consumer Groups

Consumer groups allow multiple independent readers of the same Event Hub, each maintaining their own offset. Create additional consumer groups in the Azure portal or via the [Event Hub Admin](./eventhub-admin.md) node.

> **Important:** A consumer group can only have one active reader per partition. If two Fusion workflows use the same `eventHubName` + `consumerGroup`, they will conflict. Use separate consumer groups for separate workflows.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted once per event received |
| `error` | `object` | Emitted when a processing error occurs in the subscription |

### Output Schema

```json
{
  "session": {
    "partitionId": "0"
  },
  "data": {
    "body": {
      "deviceId": "sensor-42",
      "temperature": 23.7,
      "timestamp": "2026-03-10T15:00:00.000Z"
    },
    "properties": {
      "messageRetentionInDays": 1
    },
    "systemProperties": {
      "iothub-connection-device-id": "sensor-42"
    },
    "enqueuedTimeUtc": "2026-03-10T15:00:00.123Z",
    "partitionKey": "sensor-42",
    "offset": "12345",
    "sequenceNumber": 42,
    "partitionId": "0"
  }
}
```

### Output Fields

| Field | Type | Description |
|-------|------|-------------|
| `body` | `any` | The event payload as published by the producer |
| `properties` | `object` | Custom application properties attached to the event |
| `systemProperties` | `object` | Azure-managed properties (e.g. IoT Hub metadata) |
| `enqueuedTimeUtc` | `Date` | UTC timestamp when the event was enqueued in Event Hubs |
| `partitionKey` | `string \| null` | Partition routing key set by the producer |
| `offset` | `string` | Unique event offset within the partition |
| `sequenceNumber` | `number` | Monotonically increasing sequence number within the partition |
| `partitionId` | `string` | The partition this event was read from |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example: Consume Live IoT Telemetry

Subscribe to new events from the `sensor-readings` hub using the default consumer group.

**Configuration:**
```json
{
  "connectionString": "Endpoint=sb://my-ns.servicebus.windows.net/;SharedAccessKeyName=listen;SharedAccessKey=abc123;EntityPath=sensor-readings",
  "eventHubName": "sensor-readings",
  "consumerGroup": "$Default",
  "startingPosition": "latest",
  "maxBatchSize": 10,
  "maxWaitTimeInSeconds": 3
}
```

**Sample output:**
```json
{
  "data": {
    "body": { "deviceId": "sensor-42", "temp": 23.7 },
    "enqueuedTimeUtc": "2026-03-10T15:00:00.123Z",
    "sequenceNumber": 42,
    "partitionId": "0"
  }
}
```

---

### Example: Replay All Events from Beginning

Backfill a database by reading all retained events from the beginning.

**Configuration:**
```json
{
  "connectionString": "...",
  "eventHubName": "orders",
  "consumerGroup": "backfill-consumer",
  "startingPosition": "earliest",
  "maxBatchSize": 100,
  "maxWaitTimeInSeconds": 10
}
```

> Create `backfill-consumer` first using the [Event Hub Admin](./eventhub-admin.md) node.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Sample Workflow: Event Hub → Enrich → Store

Receive sensor events, enrich with a database lookup, and insert into MongoDB.

```json
{
  "nodes": [
    {
      "id": "receive-events",
      "type": "eventhub-receive",
      "config": {
        "connectionString": "Endpoint=sb://my-ns.servicebus.windows.net/;...",
        "eventHubName": "sensor-readings",
        "consumerGroup": "$Default",
        "startingPosition": "latest"
      }
    },
    {
      "id": "enrich",
      "type": "postgres-action",
      "config": {
        "operation": "Select",
        "query": "SELECT name, location FROM devices WHERE device_id = $1",
        "params": ["{{input.data.body.deviceId}}"]
      }
    },
    {
      "id": "store",
      "type": "mongodb-action",
      "config": {
        "operation": "Insert One",
        "collection": "readings",
        "document": {
          "deviceId": "{{input.data.body.deviceId}}",
          "location": "{{input.rows[0].location}}",
          "temperature": "{{input.data.body.temperature}}",
          "receivedAt": "{{input.data.enqueuedTimeUtc}}"
        }
      }
    }
  ]
}
```

### Common Patterns

- **Event Fan-out:** Event Hub Receive → multiple parallel branches (store + alert + forward)
- **Dead-Letter Replay:** `startingPosition: earliest` + consumer group → reprocess failed events
- **Cross-Service Bridge:** Event Hub Receive → Kafka Publish (bridge Azure events to Kafka)
- **Anomaly Detection:** Event Hub Receive → Function (threshold check) → Slack alert

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Consumer group is required`

**Cause:** `consumerGroup` was left empty.

**Solution:** Use `$Default` for the default consumer group, or specify a custom one.

#### `subscription already running`

**Cause:** `run()` was called while a subscription is already active (e.g. duplicate trigger).

**Solution:** This is handled internally — the node logs a warning and skips re-subscription. Check for duplicate workflow executions.

#### Events not received after connection

**Cause:** `startingPosition` is `latest` and no new events have been published yet.

**Solution:** Set `startingPosition` to `earliest` to verify connectivity by receiving historical events, then switch back to `latest` for production.

#### `ReceiveLinkDisconnectedError`

**Cause:** The AMQP link was disconnected, typically due to a network issue or idle timeout.

**Solution:** The `EventHubConsumerClient` handles reconnects internally. If errors persist, check network stability and Event Hub throughput unit limits.

### Error Codes

| Error | Cause | Solution |
|-------|-------|----------|
| `MessagingEntityNotFoundError` | Hub or consumer group does not exist | Verify names in Azure portal |
| `UnauthorizedError` | SAS key invalid or insufficient permissions | Use a key with `Listen` permission |
| `ReceiverDisconnectedError` | Another consumer claimed the same partition in the same consumer group | Use separate consumer groups |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Event Hub Send](./eventhub-send.md) - Publish events to an Azure Event Hub
- [Event Hub Admin](./eventhub-admin.md) - Create and manage Event Hubs and consumer groups
- [Kafka Trigger](./kafka-trigger.md) - Consume events from Apache Kafka topics

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-03-10 | Initial release |

<!-- /SECTION: changelog -->
