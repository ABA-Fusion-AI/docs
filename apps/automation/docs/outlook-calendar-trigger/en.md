---
node_id: "outlook-calendar-trigger"
title: "Outlook Calendar Trigger"
description: "Poll Outlook calendar events and emit newly detected events"
category: "triggers"
subcategory: "microsoft"
version: "1.0.0"
language: "en"
last_updated: "2026-03-11"
author: "Fusion Team"
tags:
  - trigger
  - outlook
  - calendar
  - polling
related_nodes:
  - outlook-calendar
---

<!-- SECTION: header -->

# Outlook Calendar Trigger

> **Category:** Triggers | **Type:** Trigger Node

Polls Microsoft Graph events and emits unseen calendar events.

<!-- /SECTION: header -->

---

<!-- SECTION: configuration -->

## Configuration

### Authentication

Provide either:

- `accessToken`, or
- `tenantId` + `clientId` + `clientSecret`

Optional:

- `baseUrl` (default: `https://graph.microsoft.com/v1.0`)

### Parameters

| Parameter      | Type      | Required | Default | Description                                             |
| -------------- | --------- | -------- | ------- | ------------------------------------------------------- |
| `intervalMs`   | `number`  | ❌ No    | `15000` | Poll interval (min `1000`)                              |
| `emitOnStart`  | `boolean` | ❌ No    | `false` | If `false`, first poll warms cache only                 |
| `eventFilters` | `object`  | ❌ No    | —       | `startDateTime`, `endDateTime`, `top`, `skip`, `filter` |

### Output

Each emitted item is normalized to:

- `type`, `id`, `subject`
- `organizer`, `organizerEmail`
- `start`, `startTimeZone`, `end`, `endTimeZone`
- `location`
- `attendees` (name, address, type, status)
- `body`, `bodyType`
- `raw` (original event payload)

<!-- /SECTION: configuration -->
