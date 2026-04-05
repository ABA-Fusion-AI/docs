---
node_id: "service-bus-queue"
title: "Azure Service Bus - Queue"
description: "Manage Azure Service Bus queues (list/create/update/delete)"
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
  - queue
related_nodes:
  - service-bus-send
  - service-bus-receive
---

# Azure Service Bus - Queue

> **Category:** Integrations | **Type:** Action Node

Queue admin operations using `method`: `GET`, `POST`, `PATCH`, `DELETE`.

## Configuration

| Parameter          | Type     | Required | Description                             |
| ------------------ | -------- | -------- | --------------------------------------- |
| `connectionString` | `string` | ✅ Yes   | Service Bus namespace connection string |
| `method`           | `enum`   | ✅ Yes   | `GET`/`POST`/`PATCH`/`DELETE`           |
| `queue`            | `string` | ✅ Yes\* | Queue name (`POST`/`PATCH`/`DELETE`)    |
| `options`          | `object` | ❌ No    | Create/update options (`POST`/`PATCH`)  |

\* Required except for `GET`.

Output:

- `GET`: array of queue properties
- `POST`: created queue details
- `PATCH`: updated queue details
- `DELETE`: delete response
