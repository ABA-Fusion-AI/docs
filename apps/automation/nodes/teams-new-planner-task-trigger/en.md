---
node_id: "teams-new-planner-task-trigger"
title: "Microsoft Teams - On New Planner Task Trigger"
description: "Polls planner tasks and emits newly detected tasks"
category: "triggers"
subcategory: "microsoft"
version: "1.0.0"
language: "en"
last_updated: "2026-03-11"
author: "Fusion Team"
tags:
  - trigger
  - teams
  - planner
  - polling
related_nodes:
  - teams-planner
---

# Microsoft Teams - On New Planner Task Trigger

> **Category:** Triggers | **Type:** Trigger Node

Polls `/planner/plans/{planId}/tasks` and emits tasks not seen before.

Params:

- auth fields
- `planId` (required)
- `pollInterval` (seconds, default `30`)

Outputs:

- `output`: planner task object
- `error`: `{ error, reason }`
