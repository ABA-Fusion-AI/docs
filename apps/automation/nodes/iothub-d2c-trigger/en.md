---
node_id: "iothub-d2c-trigger"
title: "Azure IoT Hub - Device-to-Cloud Trigger"
description: "Receive IoT telemetry from Event Hubs-compatible endpoint"
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
  - telemetry
related_nodes:
  - iothub-c2d
---

# Azure IoT Hub - Device-to-Cloud Trigger

> **Category:** Triggers | **Type:** Trigger Node

Subscribes to IoT telemetry events via Event Hubs-compatible endpoint.

## Configuration

| Parameter          | Type     | Required | Default    |
| ------------------ | -------- | -------- | ---------- |
| `connectionString` | `string` | ✅ Yes   | —          |
| `eventHubName`     | `string` | ✅ Yes\* | —          |
| `consumerGroup`    | `string` | ❌ No    | `$Default` |
| `startingPosition` | `enum`   | ❌ No    | `latest`   |
| `isoTimestamp`     | `string` | ❌ No    | —          |

\* Required with `Endpoint=...` event hub connection strings.

Output includes: `body`, `properties`, `systemProperties`, `enqueuedTime`, `deviceId`.
Errors go to `error`.
