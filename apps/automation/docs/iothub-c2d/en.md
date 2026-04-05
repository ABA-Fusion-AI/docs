---
node_id: "iothub-c2d"
title: "Azure IoT Hub - Cloud-to-Device"
description: "Send cloud-to-device messages through IoT Hub"
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
  - c2d
related_nodes:
  - iothub-d2c-trigger
  - iothub-device-invoke
---

# Azure IoT Hub - Cloud-to-Device

> **Category:** Integrations | **Type:** Action Node

Sends a cloud-to-device message via IoT Hub service SDK.

## Configuration

| Parameter          | Type     | Required | Description                       |
| ------------------ | -------- | -------- | --------------------------------- |
| `connectionString` | `string` | ✅ Yes   | IoT Hub service connection string |
| `deviceId`         | `string` | ✅ Yes   | Target device ID                  |
| `message`          | `any`    | ✅ Yes   | Message payload                   |
| `properties`       | `object` | ❌ No    | Message properties                |
| `expirySeconds`    | `number` | ❌ No    | Expiry offset                     |

Output:
`{ messageId, status: "sent", raw }`
