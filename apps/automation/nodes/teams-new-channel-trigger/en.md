---
node_id: "teams-new-channel-trigger"
title: "Microsoft Teams - On New Channel Trigger"
description: "Polls team channels and emits newly detected channels"
category: "triggers"
subcategory: "microsoft"
version: "1.0.0"
language: "en"
last_updated: "2026-03-11"
author: "Fusion Team"
tags:
  - trigger
  - teams
  - channel
  - polling
related_nodes:
  - teams-channel
---

# Microsoft Teams - On New Channel Trigger

> **Category:** Triggers | **Type:** Trigger Node

Polls `/teams/{teamId}/channels` and emits newly seen channels.

Params:

- auth fields
- `teamId` (required)
- `pollInterval` (seconds, default `30`)

Outputs:

- `output`: channel object
- `error`: `{ error, reason }`
