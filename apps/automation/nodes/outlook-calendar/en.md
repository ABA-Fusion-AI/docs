---
node_id: "outlook-calendar"
title: "Outlook Calendar"
description: "List calendars and events, and get, create, update and delete Outlook calendar events through Microsoft Graph."
category: "integrations"
subcategory: "microsoft"
version: "1.0.0"
language: "en"
last_updated: "2026-11-23"
author: "Fusion Team"
tags:
  - integration
  - outlook
  - calendar
  - events
  - microsoft-graph
related_nodes:
  - outlook-calendar-trigger
  - outlook-mail
  - outlook-mail-trigger
---

<!-- SECTION: overview -->
# Outlook Calendar

> **Category:** Integrations &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Manage an **Outlook / Microsoft 365 calendar** through the **Microsoft Graph API v1.0**: list **calendars** and **events**, read one event, and **create**, **update** or **delete** events (with location, attendees and reminders). The node offers **6 operations**:

| Resource | Operations |
|----------|------------|
| Calendars | `getCalendars` |
| Events | `getEvents`, `getEvent`, `createEvent`, `updateEvent`, `deleteEvent` |

### Use Cases

- **Scheduling:** Create a meeting when a form is submitted or a deal reaches a stage.
- **Agenda sync:** Read upcoming events and push them to another tool.
- **Rescheduling:** Move or rename an event from a workflow.
- **Clean-up:** Delete cancelled events automatically.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Authentication

Use **one** of the two methods:

| Method | Fields | Calendar |
|--------|--------|----------|
| Access token (delegated) | `accessToken` | The signed-in user (`/me`). Leave `userId` empty. |
| Client credentials (app-only) | `tenantId`, `clientId`, `clientSecret` + `userId` | The mailbox given in `userId` (email or user ID). |

- **Access token:** a Microsoft Graph token with `Calendars.ReadWrite`. For tests you can copy one from **Graph Explorer → Access token** (it expires after about 1 hour). Works with personal Outlook.com accounts.
- **Client credentials:** an Azure app registration with the **application** permission `Calendars.ReadWrite` (admin consent required). Only works with work/school (Microsoft 365) tenants.

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `accessToken` | `string` | One auth method | Microsoft Graph bearer token. |
| `tenantId` | `string` | One auth method | Azure AD tenant ID (client credentials). |
| `clientId` | `string` | One auth method | Azure app (client) ID. |
| `clientSecret` | `string` | One auth method | Azure app client secret. |
| `baseUrl` | `string` | No | Graph base URL (default `https://graph.microsoft.com/v1.0`). |
| `userId` | `string` | With client credentials | Mailbox email or user ID. Leave empty with an access token. |
| `operation` | `enum` | Yes | Operation to perform (default `getEvents`). |
| `eventId` | `string` | For `getEvent`, `updateEvent`, `deleteEvent` | ID of the event. |
| `eventFilters` | `object` | No | `getEvents` only: `startDateTime`, `endDateTime`, `top` (1-999), `skip`, `filter` (OData `$filter`). |
| `eventData` | `object` | For `createEvent`, `updateEvent` | See below. |

#### `eventData`

| Field | Type | Description |
|-------|------|-------------|
| `subject` | `string` | Event title. **Required** for `createEvent`. |
| `body` | `string` | Description (HTML). |
| `startDateTime` | `string` | Start, ISO format without offset, e.g. `2026-11-25T10:00:00`. **Required** for `createEvent`. |
| `startTimeZone` | `string` | Time zone of the start (default `UTC`), e.g. `Africa/Casablanca`, `Europe/Paris`. |
| `endDateTime` | `string` | End, e.g. `2026-11-25T11:00:00`. **Required** for `createEvent`. |
| `endTimeZone` | `string` | Time zone of the end (default `UTC`). |
| `location` | `string` | Location name. |
| `attendees` | `string` | `createEvent` only: emails separated by commas, or a JSON array (`["a@x.com", {"email": "b@y.com", "type": "optional"}]`). Attendees receive an invitation. |
| `isAllDay` | `boolean` | `createEvent` only: all-day event (start and end must be at midnight). |
| `isReminderOn` | `boolean` | `createEvent` only: enable the reminder. |
| `reminderMinutesBeforeStart` | `number` | `createEvent` only: reminder time in minutes. |

> `updateEvent` only changes `subject`, `body`, `startDateTime`/`endDateTime` (with their time zones) and `location`, and only the fields you fill in.

> Fields are shown only for the operations that use them.

### Operations

| Operation | Graph request | Required | Returns |
|-----------|---------------|----------|---------|
| `getCalendars` | `GET /calendars` | — | `{ value: [...] }` |
| `getEvents` | `GET /events` | — | `{ value: [...] }` |
| `getEvent` | `GET /events/{id}` | `eventId` | The event. |
| `createEvent` | `POST /events` | `eventData.subject`, `startDateTime`, `endDateTime` | The created event (with its `id`). |
| `updateEvent` | `PATCH /events/{id}` | `eventId` | The updated event. |
| `deleteEvent` | `DELETE /events/{id}` | `eventId` | `{ "success": true }` |

Requests go to `/me/...` with an access token, or `/users/{userId}/...` with client credentials. Events are created in the **default calendar**.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` | Triggers the node. Use upstream values (e.g. an event `id` from the Outlook Calendar Trigger) in the parameters through expressions. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | The Microsoft Graph response, or `{ "success": true }` for `deleteEvent`. |
| `error` | `Error` | `Outlook operation failed: ...` with the Graph error (`Microsoft Graph API error (<status>): <code> - <message>`) or a missing field. |

### Output Examples

#### `getCalendars`

```json
{
  "value": [
    {
      "id": "AQMkADAwATM0MDAAMS0wMjVl...",
      "name": "Calendar",
      "isDefaultCalendar": true,
      "canEdit": true,
      "owner": { "name": "Jane", "address": "jane@example.com" }
    }
  ]
}
```

#### `createEvent`

```json
{
  "id": "AQMkADAwATM0MDAAMS0wMjVl...",
  "subject": "Project kickoff",
  "start": { "dateTime": "2026-11-25T10:00:00.0000000", "timeZone": "UTC" },
  "end": { "dateTime": "2026-11-25T11:00:00.0000000", "timeZone": "UTC" },
  "location": { "displayName": "Rabat" },
  "organizer": { "emailAddress": { "name": "Jane", "address": "jane@example.com" } },
  "webLink": "https://outlook.live.com/owa/?itemid=..."
}
```

#### `deleteEvent`

```json
{ "success": true }
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Use Outlook Calendar in a workflow
```

### How it flows

1. **Manual Trigger:** Starts the workflow.
2. **Outlook Calendar Node:** Runs the selected operation.
3. **Log Node:** Displays the response.

### Example Configurations

#### Create a meeting with attendees

```json
{
  "accessToken": "{{ secrets.MS_GRAPH_TOKEN }}",
  "operation": "createEvent",
  "eventData": {
    "subject": "Project kickoff",
    "body": "<p>Agenda: scope, planning, roles.</p>",
    "startDateTime": "2026-11-25T10:00:00",
    "startTimeZone": "Africa/Casablanca",
    "endDateTime": "2026-11-25T11:00:00",
    "endTimeZone": "Africa/Casablanca",
    "location": "Meeting room A",
    "attendees": "alice@example.com, bob@example.com",
    "isReminderOn": true,
    "reminderMinutesBeforeStart": 15
  }
}
```

#### List the next 20 events

```json
{
  "operation": "getEvents",
  "eventFilters": { "top": 20, "filter": "start/dateTime ge '2026-11-23T00:00:00'" }
}
```

#### Move an event

```json
{
  "operation": "updateEvent",
  "eventId": "AQMkADAwATM0MDAAMS0wMjVl...",
  "eventData": {
    "startDateTime": "2026-11-26T14:00:00",
    "endDateTime": "2026-11-26T15:00:00",
    "location": "Online"
  }
}
```

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: security -->
## Security

> Store the access token and the client secret in Fusion's **Secrets** vault. Do not commit them into version control.

- An app with **application** calendar permissions can edit the calendar of **any** mailbox of the tenant: restrict it with an Exchange application access policy.
- `createEvent` with attendees sends real invitations, and `deleteEvent` is permanent: protect them from repeated triggers.

<!-- /SECTION: security -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Microsoft Graph API error (401): InvalidAuthenticationToken - ...`
- **Cause:** The access token is expired (Graph Explorer tokens last about 1 hour) or invalid.
- **Solution:** Get a new token, or use client credentials so the node refreshes the token itself.

#### `Microsoft Graph API error (403): ErrorAccessDenied - ...`
- **Cause:** Missing `Calendars.Read` / `Calendars.ReadWrite` permission.
- **Solution:** Grant the permission (and admin consent for app permissions), then get a new token.

#### `Subject, start date/time, and end date/time are required`
- **Cause:** `createEvent` without `subject`, `startDateTime` or `endDateTime`.
- **Solution:** Fill in the three fields.

#### `Microsoft Graph API error (400): ... TimeZone ...`
- **Cause:** Invalid time zone name or date format.
- **Solution:** Use `YYYY-MM-DDTHH:mm:ss` and a valid time zone (`UTC`, `Europe/Paris`, ...).

#### `Microsoft Graph API error (404): ErrorItemNotFound`
- **Cause:** Wrong or already deleted `eventId`.
- **Solution:** Get a fresh ID with `getEvents`.

#### `userId (mailbox email or ID) is required when using tenantId/clientId/clientSecret`
- **Cause:** App-only tokens have no `/me`.
- **Solution:** Set `userId` to the mailbox address.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-11-23 | Full documentation: 6 operations, authentication methods, event fields, output examples, troubleshooting |

<!-- /SECTION: changelog -->
