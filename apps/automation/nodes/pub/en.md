---
node_id: "pub"
title: "Pub"
description: "Publishes data to a channel so any workflow with a matching Sub node is triggered."
category: "messaging"
subcategory: "pub-sub"
version: "1.0.0"
language: "en"
last_updated: "2026-08-18"
author: "Fusion Team"
tags:
  - pub
  - publish
  - pubsub
  - messaging
  - events
  - channels
  - inter-workflow
  - broadcast
related_nodes:
  - sub
  - manual-trigger
  - function
  - log
  - redis-action
  - kafka-publish
---

<!-- SECTION: header -->
# Pub (Publish)

> **Category:** Messaging | **Subcategory:** Pub/Sub | **Type:** Action Node

Broadcast data and event payloads to a named channel to trigger any workflow that contains a matching [Sub (Subscribe)](../sub/en.md) node.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Pub** (Publish) node acts as the event dispatcher in Fusion's internal Pub/Sub architecture. It takes incoming workflow data, serializes it, and broadcasts it across a named channel (such as `order_created`, `user-signup-event`, or `notifications.email`).

When a message is published, Fusion's event bus immediately notifies and triggers all active workflows listening to that specific channel via a **Sub** node. This enables asynchronous, one-to-many communication across independent workflows without direct coupling or hardcoded API webhooks.

### Key Features

- **Asynchronous Event Broadcasting:** Broadcasts messages instantly to one or multiple subscriber workflows simultaneously.
- **Decoupled Architecture:** Eliminates tight coupling between upstream event producers and downstream consumers.
- **Dynamic Channel Routing:** Define static channel names or evaluate dynamic expressions to route events based on data fields (e.g., `{{outputs.Function.success.eventType}}`).
- **Flexible Payload Delivery:** Publishes any valid data structure, including JSON objects, nested arrays, strings, numbers, and booleans.
- **One-to-Many Fan-Out:** A single **Pub** execution can trigger multiple different subscriber workflows running in parallel.
- **Zero Infrastructure Overhead:** Uses Fusion's built-in event bus — no external message queues or brokers (Kafka, RabbitMQ, Redis) required.

### Architecture & Data Flow

```text
┌────────────────────────────────────────────────────────┐
│               Upstream Workflow (Producer)             │
│   Incoming Request / Webhook / DB Trigger              │
│                           ↓                            │
│                 [ Transform Data ]                     │
│                           ↓                            │
│                  [ Pub Action Node ]                   │
│             Config: Channel = "order_created"          │
└───────────────────────────┬────────────────────────────┘
                            │
                            │ Broadcast Payload
                            ▼
┌────────────────────────────────────────────────────────┐
│                Fusion Event Bus Broker                 │
│               Channel: "order_created"                 │
└─────────────┬────────────────────────────┬─────────────┘
              │                            │
     Trigger Subscriber A         Trigger Subscriber B
              │                            │
              ▼                            ▼
┌──────────────────────────┐  ┌──────────────────────────┐
│  Workflow: PDF Invoicing │  │ Workflow: Email Dispatch │
│   [ Sub Trigger Node ]   │  │   [ Sub Trigger Node ]   │
│            ↓             │  │            ↓             │
│      Generate PDF        │  │       Send Receipt       │
└──────────────────────────┘  └──────────────────────────┘
```

### Use Cases

- **E-Commerce Checkout Fan-Out:** When a customer completes a purchase, publish an `order_created` event to automatically trigger inventory decrementing, PDF invoice creation, shipping notifications, and CRM updates in parallel.
- **Event-Driven Micro-Workflows:** Split complex business logic into small, maintainable workflows that interact purely through published domain events (`user.created`, `subscription.renewed`, `payment.failed`).
- **Centralized Event Dispatching:** Create unified entry-point webhooks that ingest external webhooks (Stripe, GitHub, Shopify) and publish normalized internal events to dedicated handler channels.
- **Non-Blocking Background Tasks:** Offload heavy processing (image compression, report exports, AI summaries) by publishing job data to a background processing channel, allowing the main workflow to respond immediately.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `channel` | `string` | ✅ Yes | — | The name or identifier of the channel to publish data to (e.g., `order_created`, `notifications.email`, `test_channel`). |

---

### Parameter Details

#### `channel`
The target topic/channel identifier where the payload will be broadcasted. Any workflow with a **Sub** node listening to this channel will be triggered.
- **Type:** `string`
- **Required:** Yes
- **Expression Enabled:** Yes (supports dynamic evaluation like `orders.{{outputs.Function.success.region}}`)
- **Naming Best Practices:**
  - Dot notation for hierarchical categorization: `billing.invoice.paid`, `notifications.slack`
  - Snake_case or kebab-case for simple names: `order_created`, `user-signup-event`
- **Example Values:**
  - `test_channel`
  - `order_created`
  - `notifications.email`
  - `user-signup-event`
  - `inventory.stock.depleted`

> [!TIP]
> Ensure channel names in the **Pub** node match the exact string and case used in corresponding **Sub** trigger nodes.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | The payload data to publish to the target channel. Accepts JSON objects, arrays, strings, or numbers. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `any` | Emitted when the data is successfully published to the channel. Passes through the published data or delivery confirmation. |
| `error` | `Error` | Emitted if channel validation fails or an event bus delivery error occurs. |

---

### Output Data Structure

The `success` output passes through the published payload or returns a delivery acknowledgment object:

#### Example 1: Object Payload (`order_created`)

```json
{
  "orderId": "ORD-98234",
  "customerId": "CUST-1042",
  "customerEmail": "sarah.connor@example.com",
  "currency": "USD",
  "totalAmount": 289.50,
  "items": [
    {
      "sku": "PROD-A1",
      "name": "Wireless Noise-Canceling Headphones",
      "quantity": 1,
      "price": 249.50
    },
    {
      "sku": "PROD-C4",
      "name": "Protective Travel Case",
      "quantity": 1,
      "price": 40.00
    }
  ],
  "publishedAt": "2026-08-18T14:50:00.000Z"
}
```

#### Example 2: Event Notification Payload (`notifications.email`)

```json
{
  "recipient": "dev-team@example.com",
  "subject": "Deployment Successful - Release v2.4.0",
  "template": "release-notification",
  "metadata": {
    "environment": "production",
    "deployer": "admin",
    "commitHash": "43996a5"
  }
}
```

#### Example 3: String Payload (`test_channel`)

```json
"test"
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Basic String Publishing

Publish a test string to `test_channel`.

**Workflow Configuration:**

```text
Manual Trigger
  → Function (return "test")
  → Pub (Channel: "test_channel")
```

**Result:**
Triggers all workflows containing a **Sub** node configured with `channel: "test_channel"` and passes `"test"` to them.

---

### Example 2: E-Commerce Order Event Broadcast

Broadcast an order payload after customer checkout.

**Workflow Configuration:**

```text
HTTP Webhook (Receive Order POST)
  → Function (Format order details)
  → Pub (Channel: "order_created")
```

**Downstream Subscribers:**
- Workflow A (`Sub: order_created`) → Generate Invoice PDF
- Workflow B (`Sub: order_created`) → Decrement Inventory Stock
- Workflow C (`Sub: order_created`) → Send Customer Order Confirmation

---

### Example 3: Centralized Notification Dispatcher

Publish email notification payloads to a shared messaging pipeline.

**Workflow Configuration:**

```text
Manual Trigger / Scheduled Check
  → Function (Prepare recipient & email template data)
  → Pub (Channel: "notifications.email")
```

**Downstream Subscribers:**
- Notification Hub (`Sub: notifications.email`) → Send email via Brevo / SES

---

### Example 4: Dynamic Channel Routing

Dynamically route events to different region-specific channels based on incoming data.

**Workflow Configuration:**

```text
HTTP Webhook (Incoming Lead)
  → Pub (Channel: `leads.{{outputs.Webhook.success.country}}`)
```

**Result:**
- Leads from France publish to `leads.FR`
- Leads from the United States publish to `leads.US`
- Dedicated regional subscriber workflows handle the lead assignment.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Multi-Channel Publishing Example Workflow
```

### How it flows

1. **Trigger Node:** Starts the workflow via **Manual Trigger** or external webhook.
2. **Function Node:** Prepares the payload data (e.g. returning `"test"`, `"order_num_1"`, `"hello my friend"`, or `"acces"`).
3. **Pub Node:** Publishes the data to the corresponding target channel (`test_channel`, `order_created`, `notifications.email`, `user-signup-event`), instantly firing any active workflows listening on those channels.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: best-practices -->
## Best Practices

1. **Explicit Channel Naming:** Use clear, namespaced names (`domain.action` or `service.entity.event`) to avoid naming collisions across projects.
2. **Include Timestamps & Identifiers:** Always pass unique message IDs and timestamps in the payload for easier tracing and debugging across workflows.
3. **Verify Subscriber Status:** Ensure subscriber workflows with matching **Sub** nodes are activated (running/enabled) in your workspace so they catch published events.
4. **Lightweight Payloads:** Pass entity IDs or reference keys (e.g., `{"orderId": "123"}`) if payloads are very large, allowing subscriber workflows to query database stores as needed.

<!-- /SECTION: best-practices -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Subscriber Workflows Are Not Triggered
- **Symptom:** The **Pub** node runs without errors, but target **Sub** workflows do not execute.
- **Cause 1 - Channel Name Typo:** Ensure exact spelling and case between the **Pub** node and the **Sub** node (e.g., `order_created` vs `Order_Created`).
- **Cause 2 - Subscriber Workflow Disabled:** The receiving workflow must be active/published in the workspace to listen to event channels.
- **Cause 3 - Empty Channel Name:** Verify that the `channel` parameter is not blank or evaluating to `undefined`.

#### Payload Data Missing in Subscriber
- **Symptom:** The subscriber triggers, but the incoming data is `null` or `{}`.
- **Cause:** The node preceding the **Pub** node did not output any data to pass to `input`.
- **Solution:** Verify the upstream node output in the execution execution logs and ensure the payload is properly formatted before reaching **Pub**.

---

### Error Reference

| Error Message | Cause | Solution |
|---------------|-------|----------|
| `Channel parameter is required` | The `channel` parameter was left blank | Provide a valid channel name |
| `Failed to publish to channel` | Internal event bus temporary error | Check automation engine status and retry |
| `Payload serialization error` | Payload contains circular references or non-serializable objects | Ensure payload is standard JSON-serializable data |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related Nodes

- [Sub (Subscribe)](../sub/en.md) — Listen to published channels and trigger workflows
- [Manual Trigger](../manual-trigger/en.md) — Manually trigger workflows during testing
- [Function](../function/en.md) — Construct, transform, and format payload data prior to publishing
- [Log](../log/en.md) — Print workflow output to the execution console
- [Redis Action](../redis-action/en.md) — Publish messages to external Redis channels
- [Kafka Publish](../kafka-publish/en.md) — Publish streaming event records to Apache Kafka topics

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-18 | Initial release of the Pub action node for inter-workflow channel publishing |

<!-- /SECTION: changelog -->
