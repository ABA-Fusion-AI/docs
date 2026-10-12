---
node_id: "amqp-trigger"
title: "AMQP Trigger"
description: "Triggers the workflow when a message arrives on an AMQP 1.0 address (queue or topic)."
category: "triggers-ingress"
subcategory: "messaging-event-ingress"
version: "1.0.0"
language: "en"
last_updated: "2026-10-12"
author: "Fusion Team"
tags:
  - "amqp"
  - "amqp-1.0"
  - "trigger"
  - "rabbitmq"
  - "activemq"
  - "service-bus"
  - "messaging"
  - "queues"
  - "event-driven"
related_nodes:
  - "amqp-sender"
  - "log"
  - "http-request"
  - "set-variable"
  - "function"
---

<!-- SECTION: header -->

# AMQP Trigger

> **Category:** Triggers & Ingress / Messaging & Event Ingress&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Trigger Node

Triggers workflow executions in real time whenever messages arrive on an AMQP 1.0 queue or topic address.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->

## Overview

The **AMQP Trigger** node connects to any standard **AMQP 1.0** message broker (such as RabbitMQ, Apache ActiveMQ Artemis, Azure Service Bus, or Apache Qpid) and listens continuously for incoming messages on a specified queue or topic.

When a message is published to the address, the trigger automatically captures the event, decodes the payload, extracts message properties, and emits the structured data to downstream nodes in your workflow.

### Key Features

- **Full AMQP 1.0 Protocol Compliance:** Connects to any broker supporting the standard OASIS AMQP 1.0 specification.
- **Smart Payload Normalization:** Automatically unmarshals AMQP Data sections (`typecode: 117`) and binary buffers, parsing valid JSON into clean JavaScript objects or preserving plain text strings.
- **Property Extraction:** Captures critical AMQP envelope metadata (`messageId`, `correlationId`, `contentType`, `subject`) for distributed tracing.
- **Auto-Reconnection:** Built with resilient connection recovery (`reconnect: true`) to gracefully resume listening after transient broker or network restarts.
- **TLS / SSL Encryption:** Supports secure encrypted connections (`useTls: true`, port 5671) for cloud and enterprise deployments.
- **Credential Protection:** Automatically masks sensitive authentication tokens and passwords (`[REDACTED]`) in error traces.

### Processing Flow

1. **Setup & Connection:** The node connects to the configured AMQP broker using TCP or TLS.
2. **Receiver Creation:** Upon workflow activation (`run`), it creates an AMQP receiver subscribed to the specified `address`.
3. **Message Consumption:** Incoming messages trigger the `message` event. The payload body is normalized from binary bytes to native JSON or text.
4. **Workflow Execution:** The parsed message and its headers are emitted through the `success` output port to trigger downstream nodes.
5. **Lifecycle Management:** Pausing or stopping the workflow cleanly closes the AMQP receiver and connection to prevent memory leaks and dangling sessions.

### Use Cases

- **Order & Payment Processing:** Ingest e-commerce checkout events and inventory updates in real time.
- **Microservice Integration:** Receive asynchronous commands and domain events published across distributed services.
- **Cloud to On-Premise Bridging:** Connect cloud workflows to enterprise message brokers (RabbitMQ, Azure Service Bus, ActiveMQ).
- **IoT & Telemetry:** Ingest high-throughput telemetry data published over AMQP topics.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|---|---|---|---|---|
| `host` | `string` | ✅ Yes | — | Hostname or IP address of the AMQP 1.0 broker (e.g., `amqp.example.com`, `localhost`, or `<namespace>.servicebus.windows.net`). |
| `port` | `number` | ❌ No | `5672` | TCP port of the broker (1–65535). Standard unencrypted port is `5672`; secure TLS port is typically `5671`. |
| `username` | `string` | ❌ No | — | Username for broker authentication (e.g., `guest`, or Azure SAS policy name `RootManageSharedAccessKey`). |
| `password` | `string` | ❌ No | — | Password or access key for broker authentication. |
| `useTls` | `boolean` | ❌ No | `false` | Enable TLS encryption. When enabled, connections use TLS transport (defaulting to port `5671`). |
| `address` | `string` | ✅ Yes | — | Target queue or topic address to consume messages from (e.g., `orders.queue`, `telemetry/sensor1`). |

### Broker Connection Guides

#### 1. RabbitMQ
- Ensure the **AMQP 1.0 plugin** is enabled (`rabbitmq-plugins enable rabbitmq_amqp1_0`).
- **Host:** `localhost` or your remote RabbitMQ server host.
- **Port:** `5672` (or `5671` with TLS).
- **Username / Password:** Default credentials are `guest` / `guest` (or your configured user).
- **Address:** The queue name (e.g., `test-queue`).

#### 2. Apache ActiveMQ Artemis
- ActiveMQ Artemis supports AMQP 1.0 natively on its default acceptor.
- **Host:** Broker host or IP.
- **Port:** `5672`.
- **Address:** Queue or address name configured in Artemis.

#### 3. Azure Service Bus
- **Host:** `<your-namespace>.servicebus.windows.net`.
- **Port:** `5671`.
- **Use TLS:** `true` (checked).
- **Username:** `RootManageSharedAccessKey` (or custom SAS policy name).
- **Password:** Primary Key from your Shared Access Policy.
- **Address:** Queue name or Topic subscription name.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->

## Inputs & Outputs

### Inputs

The **AMQP Trigger** is a trigger node and does not accept incoming workflow connections. It initiates workflows when an external AMQP message is received.

### Outputs

| Output | Type | Description |
|---|---|---|
| `success` | `object` | Emitted when an AMQP message is successfully received and parsed. |
| `error` | `object` | Emitted if a receiver or connection error occurs during consumption. |

### Output Schemas

#### Success Output (`success`)

When a message arrives, the node emits an object with the source address, normalized message payload, and envelope properties:

```json
{
  "address": "orders.queue",
  "message": {
    "orderId": "CMD-2026-99",
    "client": "Sarah Connor",
    "total": 149.99,
    "items": [
      {
        "sku": "PROD-A",
        "qty": 2
      }
    ]
  },
  "properties": {
    "messageId": "msg-00123",
    "contentType": "application/json",
    "correlationId": "req-98765",
    "subject": "orders.created"
  }
}
```

> **Note:** If the incoming payload is a plain text string instead of JSON, the `message` field contains the raw string value (e.g. `"Sensor value: 42.5"`).

#### Error Output (`error`)

If a connection failure or receiver error occurs, details are routed to the `error` output with credentials redacted:

```json
{
  "address": "orders.queue",
  "error": "Failed to connect to AMQP 1.0 broker: connect ECONNREFUSED 127.0.0.1:5672"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->

## Examples

### Basic JSON Message Ingestion

Listen for order creation messages and forward them to a notification service:

**Node Parameters:**
```json
{
  "host": "rabbitmq.internal",
  "port": 5672,
  "username": "app-user",
  "password": "••••••••",
  "useTls": false,
  "address": "orders.queue"
}
```

**Downstream Field Access:**
Downstream nodes (such as **HTTP Request**, **Email**, or **Function**) can directly reference the payload fields:
- Order ID: `{{ $json.message.orderId }}`
- Customer: `{{ $json.message.client }}`
- Total: `{{ $json.message.total }}`

---

### Example Workflow

The following workflow consumes messages from `orders.queue` and inspects the payload using a **Log** node.

```fusion-workflow
src: example.workflow.json
title: Ingest AMQP Messages and Log Output
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->

## Troubleshooting

### Connection Refused (`ECONNREFUSED`)
- Verify that the broker service is running and listening on the specified port.
- Check firewall rules between your Fusion deployment and the broker host.

### RabbitMQ Protocol Version Mismatch (`AMQP 0-9-1` vs `AMQP 1.0`)
- Standard RabbitMQ defaults to AMQP 0-9-1. To connect using AMQP 1.0, you must enable the plugin by running:
  ```bash
  rabbitmq-plugins enable rabbitmq_amqp1_0
  ```

### Authentication Failed
- Verify username and password credentials.
- On RabbitMQ, check that the user has read permissions on the target virtual host and queue.
- On Azure Service Bus, verify that the SAS key has **Listen** permissions on the queue.

<!-- /SECTION: troubleshooting -->
