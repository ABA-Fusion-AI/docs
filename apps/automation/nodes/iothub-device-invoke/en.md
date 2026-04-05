---
node_id: "iothub-device-invoke"
title: "Azure IoT Hub - Invoke Device"
description: "Invoke direct methods on IoT devices"
category: "integrations"
subcategory: "azure"
version: "1.0.0"
language: "en"
last_updated: "2026-03-11"
author: "Fusion Team"
tags:
  - integration
  - azure
  - iothub
  - direct-method
related_nodes:
  - iothub-c2d
---

# Azure IoT Hub - Invoke Device

> **Category:** Integrations | **Type:** Action Node

Calls a direct method on a device.

## Configuration

| Parameter          | Type     | Required | Description                       |
| ------------------ | -------- | -------- | --------------------------------- |
| `connectionString` | `string` | ✅ Yes   | IoT Hub service connection string |
| `deviceId`         | `string` | ✅ Yes   | Target device ID                  |
| `methodName`       | `string` | ✅ Yes   | Direct method name                |
| `payload`          | `any`    | ❌ No    | Method payload                    |
| `timeoutMs`        | `number` | ❌ No    | Timeout (converted to seconds)    |

Output:
`{ status, payload, raw }`
