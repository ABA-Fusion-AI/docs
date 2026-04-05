---
node_id: "teams-new-team-member-trigger"
title: "Microsoft Teams - On New Team Member Trigger"
description: "Polls team members and emits newly detected members"
category: "triggers"
subcategory: "microsoft"
version: "1.0.0"
language: "en"
last_updated: "2026-03-11"
author: "Fusion Team"
tags:
  - trigger
  - teams
  - member
  - polling
related_nodes:
  - teams-channel
---

# Microsoft Teams - On New Team Member Trigger

> **Category:** Triggers | **Type:** Trigger Node

Polls `/teams/{teamId}/members` and emits newly seen members.

Params:

- auth fields
- `teamId` (required)
- `pollInterval` (seconds, default `30`)

Outputs:

- `output`: member object
- `error`: `{ error, reason }`
