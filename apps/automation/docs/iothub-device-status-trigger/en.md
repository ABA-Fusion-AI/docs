---
node_id: "iothub-device-status-trigger"
title: "Azure IoT Hub - Device Status Trigger"
description: "Emit events when devices connect or disconnect"
category: "triggers"
subcategory: "azure"
version: "1.0.0"
language: "en"
last_updated: "2026-03-11"
author: "Fusion Team"
tags:
  - trigger
  - azure
  - iothub
  - lifecycle
related_nodes:
  - iothub-device-manager
---

# Azure IoT Hub - Device Status Trigger

> **Category:** Triggers | **Type:** Trigger Node

Consumes lifecycle events and emits `connected`/`disconnected` status.

## Configuration

| Parameter          | Type     | Required | Default    |
| ------------------ | -------- | -------- | ---------- |
| `connectionString` | `string` | ✅ Yes   | —          |
| `eventHubName`     | `string` | ✅ Yes   | —          |
| `consumerGroup`    | `string` | ❌ No    | `$Default` |
| `status`           | `enum`   | ❌ No    | `both`     |
| `deviceId`         | `string` | ❌ No    | —          |
| `retryTimeoutMs`   | `number` | ❌ No    | `20000`    |

Output includes: `deviceId`, `status`, `enqueuedTime`, `body`, `properties`.
Errors go to `error` output.
