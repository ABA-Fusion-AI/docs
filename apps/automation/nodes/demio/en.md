---
node_id: "demio"
title: "Demio"
description: "List Demio webinars, get the details and dates of an event, and register participants through the Demio REST API."
category: "peer-only"
subcategory: "integrations"
version: "1.0.0"
language: "en"
last_updated: "2026-11-24"
author: "Fusion Team"
tags:
  - demio
  - webinar
  - events
  - registration
  - marketing
related_nodes:
  - gotowebinar
---

<!-- SECTION: overview -->
# Demio

> **Category:** Peer-Only Integrations &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Manage **Demio** webinars from a workflow through the **Demio REST API v1** (`https://my.demio.com/api/v1`): list your events, read the details and dates of one event, and **register participants**. The node offers **3 operations**:

| Operation | Description |
|-----------|-------------|
| `getEvents` | List the webinars of the account. |
| `getEventDetails` | Details of one event, including its dates (`date_id`). |
| `registerParticipant` | Register a person to an event and get their personal join link. |

### Use Cases

- **Form to webinar:** Register people who fill in a form or sign up on a landing page.
- **CRM sync:** Register leads from a CRM and store their join link.
- **Reminders:** Read upcoming events and send reminders through email or chat nodes.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Authentication

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `apiKey` | `string` | Yes | Demio API key, sent as `Api-Key`. |
| `apiSecret` | `string` | Yes | Demio API secret, sent as `Api-Secret`. |

Find both in Demio under **Settings → API**. API access depends on your Demio plan.

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `operation` | `enum` | Yes | `getEvents` (default), `getEventDetails` or `registerParticipant`. |
| `eventId` | `string` | For `getEventDetails`, `registerParticipant` | **Numeric** event ID (the `id` returned by `getEvents`, e.g. `1023406`). |
| `name` | `string` | For `registerParticipant` | Participant name (at least 3 characters). |
| `email` | `string` | For `registerParticipant` | Participant email address. |

> Fields are shown only for the operations that use them. All parameters accept expressions.

### Operations

| Operation | Request | Required | Returns |
|-----------|---------|----------|---------|
| `getEvents` | `GET /events` | — | Array of events. |
| `getEventDetails` | `GET /event/{eventId}` | `eventId` | Event with its `dates`. |
| `registerParticipant` | `PUT /event/register` with `{ id, name, email }` | `eventId`, `name`, `email` | `{ hash, join_link }` |

> Registering a participant sends them Demio's confirmation email. Registering the same email twice returns the same participant.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` | Triggers the node. Use upstream values (e.g. a name and email from a form) in the parameters through expressions. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` / `array` | The Demio API response. |
| `error` | `Error` | Input errors (`A numeric eventId is required for ...`, `A valid email is required ...`) or API errors `Demio error [<status>]: ...` (keys are redacted in the message). |

### Output Examples

#### `getEvents`

```json
[
  {
    "id": 1023406,
    "labeled_name": "[Single Series] My First Webinar Experience on Demio",
    "name": "My First Webinar Experience on Demio",
    "description": null,
    "date_id": 6829290,
    "status": "scheduled",
    "timestamp": 1795525200,
    "zone": "America/New_York",
    "registration_url": "https://my.demio.com/ref/h7AaRPDfE8ybraCj",
    "automated": null
  }
]
```

#### `getEventDetails`

```json
{
  "id": 1023406,
  "name": "My First Webinar Experience on Demio",
  "description": null,
  "registrants_stay_registered": false,
  "registration_url": "https://my.demio.com/ref/h7AaRPDfE8ybraCj",
  "automated": null,
  "next_date_id": 6829290,
  "dates": [
    {
      "date_id": 6829290,
      "status": "scheduled",
      "timestamp": 1795525200,
      "datetime": "November 24th, 2026 at 8:00AM EST",
      "zone": "America/New_York"
    }
  ]
}
```

#### `registerParticipant`

```json
{
  "hash": "t6GP3yOW74BjSX1y",
  "join_link": "https://event.demio.com/simulated-webinar/t6GP3yOW74BjSX1y"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Use Demio in a workflow
```

### How it flows

1. **Manual Trigger:** Starts the workflow.
2. **Demio Node:** Runs the selected operation.
3. **Log Node:** Displays the response.

### Example Configurations

#### List events

```json
{
  "apiKey": "{{ secrets.DEMIO_API_KEY }}",
  "apiSecret": "{{ secrets.DEMIO_API_SECRET }}",
  "operation": "getEvents"
}
```

#### Register a participant from a form

```json
{
  "apiKey": "{{ secrets.DEMIO_API_KEY }}",
  "apiSecret": "{{ secrets.DEMIO_API_SECRET }}",
  "operation": "registerParticipant",
  "eventId": "1023406",
  "name": "{{ input.name }}",
  "email": "{{ input.email }}"
}
```

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: security -->
## Security

> Store the API key and secret in Fusion's **Secrets** vault. Do not commit them into version control.

- `registerParticipant` sends real emails to the participant: validate the input and protect it from repeated triggers.
- The key and secret are removed from error messages.

<!-- /SECTION: security -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Demio error [401]` / `[403]`
- **Cause:** Wrong API key or secret, or API not available on the plan.
- **Solution:** Copy them again from **Settings → API**.

#### `A numeric eventId is required for ...`
- **Cause:** `eventId` is empty or not a number (e.g. the registration URL was pasted).
- **Solution:** Use the numeric `id` from `getEvents`.

#### `name must contain at least 3 characters for registerParticipant`
- **Cause:** Name too short.
- **Solution:** Give at least 3 characters.

#### `A valid email is required for registerParticipant`
- **Cause:** Missing or malformed email.
- **Solution:** Check the email value or expression.

#### `Demio error [404]`
- **Cause:** The event does not exist or was deleted.
- **Solution:** List events with `getEvents` and use a current `id`.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-11-24 | Full documentation: 3 operations, authentication, output examples, troubleshooting |

<!-- /SECTION: changelog -->
