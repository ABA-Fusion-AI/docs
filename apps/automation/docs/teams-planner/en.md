---
node_id: "teams-planner"
title: "Microsoft Teams - Planner"
description: "Planner task operations via Microsoft Graph"
category: "integrations"
subcategory: "microsoft"
version: "1.0.0"
language: "en"
last_updated: "2026-03-11"
author: "Fusion Team"
tags:
  - integration
  - teams
  - planner
  - microsoft-graph
related_nodes:
  - teams-new-planner-task-trigger
  - teams-planner-task-updated-trigger
---

# Microsoft Teams - Planner

> **Category:** Integrations | **Type:** Action Node

Operations: `create`, `get`, `list`, `update`, `delete`.

## Configuration

Auth: `accessToken` or `tenantId` + `clientId` + `clientSecret`.

Key params:

- `planId` for `create`/`list`
- `taskId` for `get`/`update`/`delete`
- `title`, `bucketId`, `assignments` for `create`
- `title`, `percentComplete`, `assignments`, `etag` for `update`
- `top` for list

Outputs:

- Graph payload for create/get/list
- update returns payload or `{ ok: true }` when body is empty
- delete returns `{ ok: true }`
