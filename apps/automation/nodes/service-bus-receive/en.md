---
node_id: "service-bus-receive"
title: "Azure Service Bus - Receive"
description: "Receive messages from Service Bus queues or topic subscriptions"
category: "triggers"
subcategory: "azure"
version: "1.0.0"
language: "en"
last_updated: "2026-03-11"
author: "Fusion Team"
tags:
  - trigger
  - azure
  - service-bus
  - messaging
related_nodes:
  - service-bus-send
  - service-bus-subscription
---

# Azure Service Bus - Receive

> **Category:** Triggers | **Type:** Trigger Node

Subscribes to messages and emits each received message payload.

## Configuration

| Parameter              | Type      | Required | Description                                  |
| ---------------------- | --------- | -------- | -------------------------------------------- |
| `connectionString`     | `string`  | ✅ Yes   | Service Bus namespace connection string      |
| `entityType`           | `enum`    | ❌ No    | `queue` or `topic` (default `queue`)         |
| `entity`               | `string`  | ✅ Yes   | Queue/topic name                             |
| `subscription`         | `string`  | ✅ Yes\* | Topic subscription name (`entityType=topic`) |
| `receiveMode`          | `enum`    | ❌ No    | `peekLock` or `receiveAndDelete`             |
| `deadLetter`           | `boolean` | ❌ No    | Receive from dead-letter subqueue            |
| `maxConcurrentCalls`   | `number`  | ❌ No    | Concurrent message handlers (default `1`)    |
| `autoCompleteMessages` | `boolean` | ❌ No    | SDK auto-complete toggle                     |
| `prefetchCount`        | `number`  | ❌ No    | Prefetch buffer size                         |
| `sessionId`            | `string`  | ❌ No    | Receive from specific session                |
| `retryTimeoutMs`       | `number`  | ❌ No    | Retry timeout (default `20000`)              |

\* Required for topic mode.

Output message fields include:

- `body`, `applicationProperties`, `enqueuedTimeUtc`
- `messageId`, `correlationId`, `deliveryCount`
- `lockedUntilUtc`, `sequenceNumber`, `partitionKey`
- `deadLetterSource`, `subject`

Errors are emitted to `error` output.
