---
node_id: "outlook-calendar-trigger"
title: "Outlook Calendar Trigger"
description: "Poll an Outlook calendar through Microsoft Graph and start the workflow for each new event."
category: "triggers"
subcategory: "microsoft"
version: "1.0.0"
language: "en"
last_updated: "2026-11-23"
author: "Fusion Team"
tags:
  - trigger
  - outlook
  - calendar
  - events
  - polling
  - microsoft-graph
related_nodes:
  - outlook-calendar
  - outlook-mail-trigger
---

<!-- SECTION: overview -->
# Outlook Calendar Trigger

> **Category:** Triggers &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Trigger Node

Start a workflow when a **new event** appears in an **Outlook / Microsoft 365** calendar. The node **polls** Microsoft Graph (`GET /events`) at a fixed interval, remembers the events it has already seen, and emits **one event per new calendar entry** in a simple format (subject, organizer, start/end, location, attendees).

### Use Cases

- **Meeting preparation:** When a meeting is booked, gather information about the attendees and send a briefing.
- **Sync:** Copy new Outlook events to another calendar or CRM.
- **Notifications:** Post new meetings in a team channel.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Authentication

Use **one** of the two methods:

| Method | Fields | Calendar |
|--------|--------|----------|
| Access token (delegated) | `accessToken` | The signed-in user (`/me`). Leave `userId` empty. |
| Client credentials (app-only) | `tenantId`, `clientId`, `clientSecret` + `userId` | The calendar of the mailbox in `userId`. |

The token needs `Calendars.Read`. A Graph Explorer token expires after about 1 hour, so use **client credentials** for a trigger that runs for a long time.

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `accessToken` | `string` | One auth method | — | Microsoft Graph bearer token. |
| `tenantId` / `clientId` / `clientSecret` | `string` | One auth method | — | Azure app credentials (client credentials flow). |
| `baseUrl` | `string` | No | `https://graph.microsoft.com/v1.0` | Graph base URL. |
| `userId` | `string` | With client credentials | — | Mailbox email or user ID. |
| `intervalMs` | `number` | No | `15000` | Poll interval in milliseconds (minimum `1000`). |
| `emitOnStart` | `boolean` | No | `false` | `false`: the first poll only records existing events, so only events created **after** the start are emitted. `true`: existing events are emitted at start too. |
| `eventFilters` | `object` | No | — | `startDateTime`, `endDateTime`, `top` (1-999), `skip`, `filter` (OData `$filter`). |

> The trigger detects **new** events (new IDs). Changes to an existing event are not emitted.

> The list of seen events is kept in memory: after a workflow restart, the first poll starts again from `emitOnStart`.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

This node has no inputs. It starts on its own when the workflow is started.

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | One event per new calendar entry (see below). |
| `error` | `Error` | Configuration errors (missing authentication or `userId`). |

| Field | Description |
|-------|-------------|
| `type` | Always `"event"`. |
| `id` | Event ID (use it with the Outlook Calendar node: `getEvent`, `updateEvent`, `deleteEvent`). |
| `subject` | Title. |
| `organizer` / `organizerEmail` | Organizer name and address. |
| `start`, `startTimeZone`, `end`, `endTimeZone` | Dates and time zones. |
| `location` | Location name. |
| `attendees` | Array of `{ name, address, type, status }`. |
| `body`, `bodyType` | Description and `html` / `text`. |
| `raw` | The full Graph event. |

### Output Example

```json
{
  "type": "event",
  "id": "AQMkADAwATM0MDAAMS0wMjVl...",
  "subject": "Project kickoff",
  "organizer": "Jane Doe",
  "organizerEmail": "jane@example.com",
  "start": "2026-11-25T10:00:00.0000000",
  "startTimeZone": "UTC",
  "end": "2026-11-25T11:00:00.0000000",
  "endTimeZone": "UTC",
  "location": "Rabat",
  "attendees": [{ "name": "Bob", "address": "bob@example.com", "type": "required", "status": "none" }],
  "body": "<html><body>Agenda ...</body></html>",
  "bodyType": "html",
  "raw": { "...": "..." }
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Listen for new Outlook calendar events
```

### How it flows

1. **Outlook Calendar Trigger:** Polls the calendar and emits each new event.
2. **Log Node:** Displays the event.

### Example Configurations

#### Every new event, every minute

```json
{
  "accessToken": "{{ secrets.MS_GRAPH_TOKEN }}",
  "intervalMs": 60000,
  "emitOnStart": false
}
```

#### Watch a shared calendar (app-only)

```json
{
  "tenantId": "00000000-0000-0000-0000-000000000000",
  "clientId": "11111111-1111-1111-1111-111111111111",
  "clientSecret": "{{ secrets.AZURE_CLIENT_SECRET }}",
  "userId": "meeting-room@contoso.com",
  "eventFilters": { "top": 50 }
}
```

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: security -->
## Security

> Store the access token and the client secret in Fusion's **Secrets** vault. Do not commit them into version control.

- Event titles and descriptions are untrusted input: validate them before using them in prompts or commands.
- Keep `intervalMs` reasonable (15 s or more) to stay within Microsoft Graph throttling limits.

<!-- /SECTION: security -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### The trigger never fires
- **Cause:** `emitOnStart` is `false`, so existing events are ignored, or the token expired (poll errors are only logged).
- **Solution:** Create a new event after starting the workflow, and check that the token is still valid.

#### An updated event is not emitted
- **Cause:** Only new event IDs are detected.
- **Solution:** Expected behaviour; use the Outlook Calendar node with `getEvents` to read changes.

#### Some events are missed
- **Cause:** More new events than returned by one poll (10 by default).
- **Solution:** Increase `eventFilters.top` or shorten `intervalMs`.

#### `Either accessToken or (tenantId, clientId, clientSecret) must be provided`
- **Cause:** No authentication configured.
- **Solution:** Fill in `accessToken`, or the three client credentials fields (with `userId`).

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-11-23 | Full documentation: polling behaviour, filters, output format, troubleshooting |

<!-- /SECTION: changelog -->
