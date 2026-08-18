---
node_id: "sub"
title: "Sub"
description: "Triggers this workflow when another workflow publishes to the selected channel."
category: "triggers"
subcategory: "messaging"
version: "1.0.0"
language: "en"
last_updated: "2026-08-18"
author: "Fusion Team"
tags:
  - sub
  - subscribe
  - pubsub
  - trigger
  - messaging
  - events
  - channels
  - inter-workflow
related_nodes:
  - pub
  - manual-trigger
  - redis-subscribe
  - kafka-trigger
  - function
  - log
---

<!-- SECTION: header -->
# Sub (Subscribe)

> **Category:** Triggers | **Subcategory:** Messaging | **Type:** Trigger Node

Listen to internal Pub/Sub channels and automatically trigger workflow execution whenever another workflow or system event publishes data to the subscribed channel.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Sub** (Subscribe) node enables event-driven, decoupled inter-workflow communication within Fusion Automation. It functions as a real-time message listener that monitors a designated event channel and starts workflow execution immediately upon receiving a published payload.

By pairing the **Sub** trigger node with the **Pub** (Publish) action node, you can construct modular, microservice-style automation architectures. Rather than building massive monolithic workflows, you can decouple complex logic into specialized workflows that communicate seamlessly via lightweight named channels (such as `order_created`, `user-signup-event`, or `notifications.email`).

### Key Features

- **Real-Time Event Triggering:** Instantly activates workflow execution as soon as a message is broadcasted to the target channel.
- **Decoupled Architecture:** Breaks large, monolithic workflows into independent, maintainable, and reusable micro-workflows.
- **Dynamic Channel Expressions:** Supports static channel names or dynamic expression evaluation.
- **Arbitrary Payload Support:** Accepts any JSON payload structure, including objects, arrays, primitives, and nested event records.
- **Multi-Subscriber Support:** Multiple **Sub** nodes across different workflows can listen to the exact same channel simultaneously (one-to-many event broadcasting).
- **Zero External Infrastructure:** Operates natively within Fusion's internal event bus without requiring external brokers like Redis, Kafka, or RabbitMQ.

### Architecture & Data Flow

```text
┌──────────────────────────────────────┐
│       Publisher Workflow             │
│   (e.g., Checkout / Ingestion Flow)   │
│                  ↓                   │
│       [ Pub Node (Publish) ]         │
│     Channel: "order_created"         │
└──────────────────┬───────────────────┘
                   │
                   │ Broadcast Event Payload
                   ▼
┌────────────────────────────────────────────────────────┐
│              Fusion Event Bus Channel                  │
│               Channel: "order_created"                 │
└────────────┬─────────────────────────────┬─────────────┘
             │                             │
             ▼                             ▼
┌─────────────────────────┐   ┌─────────────────────────┐
│ Subscriber Workflow A   │   │ Subscriber Workflow B   │
│   [ Sub Trigger Node ]  │   │   [ Sub Trigger Node ]  │
│            ↓            │   │            ↓            │
│  Generate Invoice PDF   │   │  Send Customer Email    │
└─────────────────────────┘   └─────────────────────────┘
```

### Use Cases

- **Order Processing & Fulfillment:** An e-commerce checkout workflow publishes an `order_created` event; separate subscriber workflows handle inventory updates, PDF invoice generation, and shipping label creation independently.
- **Centralized Notification Service:** Multiple workflows publish alerts to a `notifications.email` or `notifications.slack` channel, routed to a dedicated workflow that handles templating, rate-limiting, and dispatch.
- **User Lifecycle Events:** Broadcast `user-signup-event` to trigger simultaneous welcome email campaigns, CRM contact creation, and analytics tracking.
- **Async Task Offloading:** Offload heavy or time-consuming operations (video transcoding, large report generation, data syncing) to background subscriber workflows without blocking the primary user-facing flow.
- **Microservice-Style Modularity:** Maintain cleaner, domain-specific workflows where each team or process owns distinct subscriber pipelines.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `channel` | `string` | ✅ Yes | — | The name or identifier of the Pub/Sub channel to listen to (e.g., `order_created`, `notifications.email`, `test_channel`). |

---

### Parameter Details

#### `channel`
The unique identifier of the event topic/channel that this trigger listens to.
- **Type:** `string`
- **Required:** Yes
- **Expression Enabled:** Yes (supports dynamic evaluation if needed)
- **Naming Conventions:**
  - Dot notation for hierarchical namespacing: `ecommerce.orders.created`, `notifications.sms`
  - Kebab-case or snake_case for simple topics: `order-created`, `user_signup_event`, `test_channel`
- **Example Values:**
  - `test_channel`
  - `order_created`
  - `user-signup-event`
  - `notifications.email`
  - `billing.invoice.generated`

> [!TIP]
> Use namespaced channel naming conventions (e.g., `domain.entity.action` like `crm.deal.won`) across your workspace to prevent accidental channel collisions between unrelated workflows.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

As a **Trigger Node**, the **Sub** node has no workflow data input ports. It fires autonomously when an upstream publisher broadcasts a payload to the subscribed channel.

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `any` | Emitted when a message is received on the configured channel. Passes the published payload data directly to downstream nodes. |
| `error` | `Error` | Emitted if an internal event-bus subscription error occurs. |

---

### Output Data Structure

The `success` output passes whatever payload was supplied by the publisher node.

#### Example 1: Object Payload (`order_created`)

When the publisher broadcasts a JSON object:

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
  "createdAt": "2026-08-18T14:45:00.000Z"
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

#### Example 3: Simple String / Primitive Payload (`test_channel`)

```json
"Hello from Publisher Workflow"
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Basic String Event Processing

Listen on `test_channel` and append validation text using a **Function** node.

**Parameter Configuration:**

```text
Channel: test_channel
```

**Workflow Flow:**
1. **Sub (`test_channel`):** Receives string input (e.g. `"hello"`).
2. **Function:** Appends `"1"` or transforms the text (`return input + "1"`).
3. **Log:** Displays `"hello1"` in the console.

---

### Example 2: E-Commerce Order Fulfillment

Trigger order processing whenever an order is submitted.

**Parameter Configuration:**

```text
Channel: order_created
```

**Downstream Pipeline:**
```text
Sub (Channel: "order_created")
  → Function (Calculate tax & discounts)
  → PDF Generator (Create invoice document)
  → Email Send (Send receipt to customerEmail)
  → Log (Record successful fulfillment)
```

---

### Example 3: User Signup Onboarding

Trigger user onboarding workflow on account registration.

**Parameter Configuration:**

```text
Channel: user-signup-event
```

**Downstream Pipeline:**
```text
Sub (Channel: "user-signup-event")
  → Function (Format contact record)
  → HubSpot / CRM Action (Create Contact)
  → Slack Action (Post new user alert to #growth channel)
  → Log
```

---

### Example 4: Notification Microservice

A centralized email dispatch workflow triggered by various microservices.

**Parameter Configuration:**

```text
Channel: notifications.email
```

**Downstream Pipeline:**
```text
Sub (Channel: "notifications.email")
  → If/Else (Check if recipient is valid)
  → Brevo / SendGrid / SES Action (Send transactional email)
  → Log (Record message ID)
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Multi-Channel Subscriptions Example Workflow
```

### How it flows

1. **Sub Node:** Listens to designated channels such as `test_channel`, `order_created`, `notifications.email`, and `user-signup-event`.
2. **Function Node:** Processes and enriches the incoming payload (e.g., appending tags, formatting strings, or preparing data structures).
3. **Log Node:** Prints the processed output to the execution console for real-time observability.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: best-practices -->
## Best Practices

1. **Consistent Channel Naming:** Establish a standard naming scheme across your team (e.g., `domain.entity.event` like `billing.subscription.canceled`).
2. **Idempotent Handlers:** Design downstream subscriber workflows to be idempotent so that receiving duplicate events does not corrupt data.
3. **Dedicated Error Handling:** Connect the `error` output of the **Sub** node or downstream nodes to alerting channels (Slack, PagerDuty, or Email).
4. **Payload Versioning:** If events change over time, include a `version` field in published payloads (e.g., `{"version": "1.0", "data": {...}}`) to maintain backward compatibility.

<!-- /SECTION: best-practices -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Workflow Does Not Trigger When Event is Published
- **Symptom:** Upstream **Pub** node completes successfully, but the **Sub** workflow never starts.
- **Cause 1 - Channel Name Mismatch:** Case sensitivity or typos in the channel name (e.g., `Order_Created` vs `order_created`). Ensure exact string matching.
- **Cause 2 - Workflow Inactive / Draft Status:** The subscriber workflow is turned off or not published in the target environment.
- **Cause 3 - Workspace Isolation:** The publishing and subscribing workflows are located in different tenant workspaces.

#### Unexpected Payload Structure in Downstream Nodes
- **Symptom:** Downstream nodes fail with `Cannot read property of undefined`.
- **Cause:** The publisher sent a different data shape than expected (e.g., a primitive string instead of an object).
- **Solution:** Add a **Function** node immediately following the **Sub** node to validate and normalize incoming payloads before processing.

---

### Error Reference

| Error Message | Cause | Solution |
|---------------|-------|----------|
| `Channel is required` | The `channel` parameter was left empty | Specify a valid channel name string |
| `Event bus connection timeout` | Internal message broker temporary interruption | Verify automation engine status and retry |
| `Invalid payload encoding` | Payload corrupted during serialization | Ensure the publishing node passes valid JSON-serializable data |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related Nodes

- [Pub](../pub/en.md) — Publish events and payloads to a named channel
- [Manual Trigger](../manual-trigger/en.md) — Manually trigger workflows for testing
- [Redis Subscribe](../redis-subscribe/en.md) — Listen to external Redis Pub/Sub channels
- [Kafka Trigger](../kafka-trigger/en.md) — Consume event messages from Apache Kafka topics
- [Function](../function/en.md) — Transform and reshape incoming event payloads
- [Log](../log/en.md) — Output received channel data to the execution console

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-18 | Initial release of the Sub trigger node with support for named Pub/Sub event channels and real-time inter-workflow communication |

<!-- /SECTION: changelog -->
