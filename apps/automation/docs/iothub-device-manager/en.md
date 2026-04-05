---
node_id: "iothub-device-manager"
title: "Azure IoT Hub - Device Manager"
description: "Manage IoT Hub device identities and twins"
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
  - device
related_nodes:
  - iothub-device-status-trigger
  - iothub-c2d
  - iothub-device-invoke
---

# Azure IoT Hub - Device Manager

> **Category:** Integrations | **Type:** Action Node

Operations:

- `createDevice`, `deleteDevice`, `getDevice`, `listDevices`, `updateDevice`
- `getTwin`, `updateTwin`, `queryDevices`

## Configuration

Required:

- `connection.connectionString`
- `operation`

Operation-specific:

- `device` for create/update
- `deviceId` for get/delete/getTwin
- `query` for `queryDevices`
- `twinPatch` for `updateTwin`
- `pageSize` for list/query

Notes:

- SAS keys must be valid 32-byte base64 for device auth.
- `updateTwin` applies patch under reported properties.
