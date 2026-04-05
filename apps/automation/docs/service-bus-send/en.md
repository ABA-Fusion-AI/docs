---
node_id: "service-bus-send"
title: "Azure Service Bus - Send"
description: "Send messages to Service Bus queues or topics"
category: "integrations"
subcategory: "azure"
version: "1.0.0"
language: "en"
last_updated: "2026-03-11"
author: "Fusion Team"
tags:
  - integration
  - azure
  - service-bus
  - messaging
related_nodes:
  - service-bus-queue
  - service-bus-topic
  - service-bus-receive
---

# Azure Service Bus - Send

> **Category:** Integrations | **Type:** Action Node

Sends one message to a queue or topic.

## Configuration

| Parameter                              | Type     | Required | Description                               |
| -------------------------------------- | -------- | -------- | ----------------------------------------- |
| `connectionString`                     | `string` | ✅ Yes   | Service Bus namespace connection string   |
| `entityType`                           | `enum`   | ❌ No    | `queue` or `topic` (default `queue`)      |
| `entity`                               | `string` | ✅ Yes   | Queue/topic name                          |
| `messageBody`                          | `any`    | ❌ No    | Message body; falls back to incoming data |
| `applicationProperties`                | `object` | ❌ No    | Custom application properties             |
| `sessionId`                            | `string` | ❌ No    | Session id (queue)                        |
| `partitionKey`                         | `string` | ❌ No    | Partition key (topic)                     |
| `messageId`/`correlationId`            | `string` | ❌ No    | Message identifiers                       |
| `timeToLiveMs`                         | `number` | ❌ No    | Message TTL in ms                         |
| `scheduledEnqueueTimeUtc`              | `string` | ❌ No    | Scheduled enqueue datetime                |
| `replyTo`/`to`/`subject`/`contentType` | `string` | ❌ No    | Extra broker message fields               |

Output:

```json
{ "status": "sent", "messageId": "..." }
```
