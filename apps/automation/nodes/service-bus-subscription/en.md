---
node_id: "service-bus-subscription"
title: "Azure Service Bus - Subscription"
description: "Manage Service Bus topic subscriptions (list/create/update/delete)"
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
  - subscription
related_nodes:
  - service-bus-topic
  - service-bus-receive
---

# Azure Service Bus - Subscription

> **Category:** Integrations | **Type:** Action Node

Subscription admin operations on a topic with `method`: `GET`, `POST`, `PATCH`, `DELETE`.

## Configuration

| Parameter          | Type     | Required | Description                                 |
| ------------------ | -------- | -------- | ------------------------------------------- |
| `connectionString` | `string` | ✅ Yes   | Service Bus namespace connection string     |
| `topic`            | `string` | ✅ Yes   | Topic name                                  |
| `method`           | `enum`   | ✅ Yes   | `GET`/`POST`/`PATCH`/`DELETE`               |
| `subscription`     | `string` | ✅ Yes\* | Subscription name (`POST`/`PATCH`/`DELETE`) |
| `options`          | `object` | ❌ No    | Create/update options                       |

\* Required except for `GET`.

Output:

- `GET`: array of subscription properties for topic
- `POST`: created subscription details
- `PATCH`: updated subscription details
- `DELETE`: delete response
