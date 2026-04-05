---
node_id: "teams-planner-task-updated-trigger"
title: "Microsoft Teams - On Planner Task Updated Trigger"
description: "Polls planner tasks and emits tasks when lastModifiedDateTime changes"
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

# Microsoft Teams - On Planner Task Updated Trigger

> **Category:** Triggers | **Type:** Trigger Node

Polls tasks in a plan and emits a task when `lastModifiedDateTime` changes.

Params:

- auth fields
- `planId` (required)
- `pollInterval` (seconds, default `30`)

Outputs:

- `output`: updated planner task object
- `error`: `{ error, reason }`
