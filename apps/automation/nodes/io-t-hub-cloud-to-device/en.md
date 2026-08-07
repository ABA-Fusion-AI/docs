---
node_id: "io-t-hub-cloud-to-device"
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
  - io-t-hub-device-to-cloud-trigger
  - io-t-hub-device-invoke
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
