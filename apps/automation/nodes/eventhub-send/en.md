---
node_id: "eventhub-send"
title: "Event Hub Send"
description: "Publish events to an Azure Event Hub using the producer client — supports custom partition keys, message properties, and dynamic event body"
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
  - producer
  - action
related_nodes:
  - eventhub-receive
  - eventhub-admin
  - kafka-publish
  - rabbitmq-action
---

<!-- SECTION: header -->
# Event Hub Send

> **Category:** Integrations | **Type:** Action Node

Publish a single event to an Azure Event Hub. Supports optional partition key routing, custom message properties, and dynamic body resolution from upstream workflow data.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Event Hub Send** node uses the `@azure/event-hubs` `EventHubProducerClient` to send events to an Azure Event Hub. It creates a batch on each execution, adds the event, and sends the batch atomically. The producer client is initialised once at `setup()` and reused across executions.

If `body` is not explicitly configured, the node forwards the **entire incoming upstream data** as the event body — making it easy to relay workflow state directly into Event Hubs without a mapping step.

### Key Features

- **Batch-Based Sending:** Uses `createBatch()` + `sendBatch()` for reliable delivery
- **Partition Key Support:** Route related events to the same partition for ordered processing
- **Dynamic Body:** When `body` is omitted, the upstream input is used as the event payload
- **Custom Properties:** Attach metadata such as retention or partition count to the message
- **Expression Support:** `body` and `partitionKey` accept dynamic expressions

### Use Cases

- Ingest workflow results (e.g. ML inference outputs) into an Event Hub for downstream stream processing
- Forward webhook payloads from external services into Azure Event Hubs for fan-out consumers
- Publish IoT-style telemetry from scheduled Fusion workflows into a real-time processing pipeline
- Bridge data from other Fusion nodes (Kafka, RabbitMQ, databases) into Azure's event streaming ecosystem

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `connectionString` | `string` | ✅ Yes | — | Azure Event Hub connection string (Shared Access Signature format) |
| `eventHubName` | `string` | ✅ Yes | — | Name of the Event Hub to publish to |
| `body` | `any` | ❌ No | *(upstream input)* | Event body to send. If omitted, the upstream workflow data is used |
| `partitionKey` | `string` | ❌ No | — | Partition key — events with the same key are routed to the same partition |
| `properties` | `object` | ❌ No | — | Optional message-level metadata (see structure below) |

### `properties` Structure

```json
{
  "messageRetentionInDays": 1,
  "partitionCount": 1
}
```

### Connection String Format

The connection string must follow the SAS format provided by the Azure portal:

```
Endpoint=sb://<namespace>.servicebus.windows.net/;SharedAccessKeyName=<policy>;SharedAccessKey=<key>;EntityPath=<hub>
```

You can also use a namespace-level connection string and specify `eventHubName` separately.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Upstream data — used as event body if `body` is not configured |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `output` | `object` | Confirmation of the sent event |
| `error` | `Error` | Emitted if the send fails or the batch is too large |

### Output Schema

```json
{
  "status": "sent",
  "body": {
    "orderId": "ORD-999",
    "total": 249.99,
    "timestamp": "2026-03-10T15:00:00.000Z"
  }
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example: Forward Upstream Data as Event

No `body` configured — the upstream workflow result is published directly.

**Configuration:**
```json
{
  "connectionString": "Endpoint=sb://my-ns.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=abc123;EntityPath=telemetry",
  "eventHubName": "telemetry"
}
```

---

### Example: Publish with Custom Body and Partition Key

Route all events for a specific device to the same partition.

**Configuration:**
```json
{
  "connectionString": "Endpoint=sb://my-ns.servicebus.windows.net/;...",
  "eventHubName": "device-telemetry",
  "partitionKey": "{{input.deviceId}}",
  "body": {
    "deviceId": "{{input.deviceId}}",
    "temperature": "{{input.temperature}}",
    "timestamp": "{{input.timestamp}}"
  }
}
```

**Output:**
```json
{
  "status": "sent",
  "body": {
    "deviceId": "sensor-42",
    "temperature": 23.7,
    "timestamp": "2026-03-10T15:00:00.000Z"
  }
}
```

---

### Example: Relay a Webhook Payload

Receive an HTTP webhook and forward the raw payload to Event Hubs.

```json
{
  "connectionString": "...",
  "eventHubName": "webhooks",
  "body": "{{input.body}}"
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Sample Workflow: Scheduled ETL → Event Hub → Stream Analytics

Run a scheduled database query and publish each row as an event for stream processing.

```json
{
  "nodes": [
    {
      "id": "schedule",
      "type": "cron",
      "config": { "expression": "*/5 * * * *" }
    },
    {
      "id": "query-db",
      "type": "postgres-action",
      "config": {
        "operation": "Select",
        "query": "SELECT * FROM sensor_readings WHERE processed = false"
      }
    },
    {
      "id": "publish-events",
      "type": "eventhub-send",
      "config": {
        "connectionString": "Endpoint=sb://my-ns.servicebus.windows.net/;...",
        "eventHubName": "sensor-readings",
        "partitionKey": "{{input.deviceId}}"
      }
    }
  ]
}
```

### Common Patterns

- **Webhook Bridge:** HTTP webhook trigger → Event Hub Send (fan-out to multiple consumers)
- **DB Change Capture:** Polling DB → Event Hub Send (feed into stream analytics)
- **Cross-Cloud Bridge:** Kafka Trigger → Event Hub Send (sync topics to Azure)
- **Alert Routing:** Function (detect anomaly) → Event Hub Send (feed alerting pipeline)

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Event Hub client not initialized`

**Cause:** `handleTick()` was called before `setup()` completed.

**Solution:** This is managed by the framework. If it occurs, verify there are no errors in `setup()` — typically caused by an invalid `connectionString`.

#### `Event too large to fit into batch`

**Cause:** The serialized event body exceeds the Event Hub batch size limit (1 MB by default).

**Solution:** Reduce the event payload. Consider splitting large objects and publishing them as multiple events, or move large binary data to blob storage and reference it by URL.

#### `Connection refused` / `AMQP transport error`

**Cause:** The connection string is invalid, or the Event Hub namespace is unreachable.

**Solution:** Verify the SAS connection string, check that the namespace exists, and confirm outbound TCP on port 5671 or 443 (AMQP over WebSocket) is allowed.

### Error Codes

| Error | Cause | Solution |
|-------|-------|----------|
| `MessagingEntityNotFoundError` | `eventHubName` does not exist | Check the hub name in the Azure portal |
| `UnauthorizedError` | SAS key expired or invalid | Regenerate the SAS key in Azure portal |
| `QuotaExceededError` | Hub is at capacity | Scale up the pricing tier or add partitions |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Event Hub Receive](./eventhub-receive.md) - Consume events from an Event Hub in real time
- [Event Hub Admin](./eventhub-admin.md) - Create and manage Event Hubs and consumer groups
- [Kafka Publish](./kafka-publish.md) - Publish messages to Apache Kafka topics

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-03-10 | Initial release |

<!-- /SECTION: changelog -->
