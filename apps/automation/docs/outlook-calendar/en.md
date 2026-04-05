---
node_id: "outlook-calendar"
title: "Outlook Calendar"
description: "Microsoft Outlook calendar operations via Microsoft Graph"
category: "integrations"
subcategory: "microsoft"
version: "1.0.0"
language: "en"
last_updated: "2026-03-11"
author: "Fusion Team"
tags:
  - integration
  - outlook
  - calendar
  - microsoft-graph
related_nodes:
  - outlook-calendar-trigger
  - outlook-mail
---

<!-- SECTION: header -->

# Outlook Calendar

> **Category:** Integrations | **Type:** Action Node

Calendar operations: list calendars/events, get event, create/update/delete event.

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

### Operations

`operation` supports:

- `getCalendars`
- `getEvents`
- `getEvent`
- `createEvent`
- `updateEvent`
- `deleteEvent`

### Key Parameters

| Parameter                 | Used By                                  | Notes                                                   |
| ------------------------- | ---------------------------------------- | ------------------------------------------------------- |
| `eventId`                 | `getEvent`, `updateEvent`, `deleteEvent` | Required                                                |
| `eventFilters`            | `getEvents`                              | `startDateTime`, `endDateTime`, `top`, `skip`, `filter` |
| `eventData.subject`       | `createEvent`                            | Required                                                |
| `eventData.startDateTime` | `createEvent`                            | Required                                                |
| `eventData.endDateTime`   | `createEvent`                            | Required                                                |
| `eventData.startTimeZone` | `createEvent`, `updateEvent`             | Default `UTC`                                           |
| `eventData.endTimeZone`   | `createEvent`, `updateEvent`             | Default `UTC`                                           |
| `eventData.attendees`     | `createEvent`                            | Comma-separated attendee values parsed by node          |

### Outputs

- `getCalendars`, `getEvents`, `getEvent`, `createEvent`, `updateEvent`: Graph response payload
- `deleteEvent`: `{ "success": true }`

<!-- /SECTION: configuration -->
