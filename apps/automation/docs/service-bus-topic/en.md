---
node_id: "service-bus-topic"
title: "Azure Service Bus - Topic"
description: "Manage Azure Service Bus topics (list/create/update/delete)"
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
  - topic
related_nodes:
  - service-bus-subscription
  - service-bus-send
  - service-bus-receive
---

# Azure Service Bus - Topic

> **Category:** Integrations | **Type:** Action Node

Topic admin operations using `method`: `GET`, `POST`, `PATCH`, `DELETE`.

## Configuration

| Parameter          | Type     | Required | Description                             |
| ------------------ | -------- | -------- | --------------------------------------- |
| `connectionString` | `string` | ✅ Yes   | Service Bus namespace connection string |
| `method`           | `enum`   | ✅ Yes   | `GET`/`POST`/`PATCH`/`DELETE`           |
| `topic`            | `string` | ✅ Yes\* | Topic name (`POST`/`PATCH`/`DELETE`)    |
| `options`          | `object` | ❌ No    | Create/update options (`POST`/`PATCH`)  |

\* Required except for `GET`.

Output:

- `GET`: array of topic properties
- `POST`: created topic details
- `PATCH`: updated topic details
- `DELETE`: delete response
