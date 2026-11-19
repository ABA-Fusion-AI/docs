---
node_id: "pagerduty"
title: "PagerDuty"
description: "Manage PagerDuty incidents, alerts, services and users: create, acknowledge and resolve incidents, and send alerts through the Events API v2."
category: "peer-only"
subcategory: "Incident Management"
version: "1.0.0"
language: "en"
last_updated: "2026-11-17"
author: "Fusion Team"
tags:
  - pagerduty
  - incidents
  - alerting
  - on-call
  - devops
  - monitoring
related_nodes:
  - circle-ci
  - smtp
  - xmpp
  - http-request
---

<!-- SECTION: overview -->
# PagerDuty

> **Category:** Peer-Only Integrations &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Manage **PagerDuty** from a Fusion workflow. The node uses the PagerDuty REST API v2 (`https://api.pagerduty.com`) for incidents, services and users, and the **Events API v2** (`https://events.pagerduty.com/v2/enqueue`) to send alerts. It offers **9 operations**:

| Resource | Operations |
|----------|------------|
| Incidents | `listIncidents`, `getIncident`, `createIncident`, `acknowledgeIncident`, `resolveIncident` |
| Alerts | `listAlerts`, `createAlert` |
| Services | `listServices` |
| Users | `listUsers` |

### Use Cases

- **Monitoring alerts:** Send an alert to PagerDuty when a workflow detects an error, a failed job or a threshold breach.
- **Incident automation:** Acknowledge or resolve incidents automatically when the problem is fixed.
- **Reporting:** List open incidents and their services for a daily on-call summary.
- **Escalation:** Create a high-urgency incident from a form, a ticket or a chat message.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `apiKey` | `string` | Yes | — | PagerDuty REST API key. |
| `fromEmail` | `string` | For create / acknowledge / resolve | — | Email of a PagerDuty user of the account, sent in the `From` header. |
| `operation` | `enum` | Yes | `listIncidents` | Operation to perform. |
| `incidentId` | `string` | For `getIncident`, `acknowledgeIncident`, `resolveIncident`, `listAlerts` | — | Incident ID (e.g. `Q2BRQE0R85WUUY`, not the incident number). |
| `title` | `string` | For `createIncident` (or `createAlert`) | — | Incident title, or alert summary. |
| `serviceId` | `string` | For `createIncident` | — | Service ID (e.g. `POOX2DQ`). |
| `urgency` | `enum` | No | `high` | `high` or `low` (`createIncident`). |
| `body` | `string` | No | — | Incident details (`createIncident`). |
| `limit` | `number` | No | `20` | Number of results for the `list…` operations. |
| `alertBody` | `string` | No | — | Alert details; used as summary when `title` is empty (`createAlert`). |
| `routingKey` | `string` | For `createAlert` | — | Integration key of an **Events API V2** integration on the service. |
| `severity` | `enum` | No | `error` | `critical`, `error`, `warning` or `info` (`createAlert`). |

> All parameters are shown for every operation in the builder; only the ones listed above are used by each operation.

### Operations

| Operation | API | Required |
|-----------|-----|----------|
| `listIncidents` | `GET /incidents` (open only: `triggered` and `acknowledged`) | `apiKey` |
| `getIncident` | `GET /incidents/{id}` | `incidentId` |
| `createIncident` | `POST /incidents` | `title`, `serviceId`, `fromEmail` |
| `acknowledgeIncident` | `PUT /incidents/{id}` (status `acknowledged`) | `incidentId`, `fromEmail` |
| `resolveIncident` | `PUT /incidents/{id}` (status `resolved`) | `incidentId`, `fromEmail` |
| `listAlerts` | `GET /incidents/{id}/alerts` | `incidentId` |
| `listServices` | `GET /services` | `apiKey` |
| `listUsers` | `GET /users` | `apiKey` |
| `createAlert` | Events API v2 `POST /v2/enqueue` (`trigger`) | `routingKey`, `title` or `alertBody` |

> **createIncident vs createAlert:** `createIncident` creates an incident directly with the REST API. `createAlert` sends an event to the service's integration; PagerDuty creates (or groups it into) an incident. Incidents created with `createIncident` have no alerts, so `listAlerts` returns an empty list for them.

### Authentication

**REST API key** (`apiKey`)
1. In PagerDuty, open **Integrations → API Access Keys**.
2. Click **Create New API Key** (leave *Read-only* unchecked to create and update incidents).
3. Copy the key (shown only once) and store it in Fusion Secrets.

**Routing key** (`routingKey`, for `createAlert`)
1. Open **Services → your service → Integrations → Add integration**.
2. Choose **Events API V2** and copy the **Integration Key**.

### Where to find the IDs

| ID | How to get it |
|----|---------------|
| `serviceId` | `listServices`, or the service URL `…/service-directory/<serviceId>` |
| `incidentId` | `listIncidents` / `createIncident`, or the incident URL `…/incidents/<incidentId>` |
| `fromEmail` | `listUsers` (email of any user of the account) |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` | Triggers the node. Use upstream values in the parameters through expressions. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | JSON response of PagerDuty. |
| `error` | `Error` | Emitted on missing fields or API errors. |

### Output Examples

#### `createIncident` / `resolveIncident`

```json
{
  "incident": {
    "id": "Q2BRQE0R85WUUY",
    "incident_number": 3,
    "title": "Server down",
    "status": "resolved",
    "urgency": "low",
    "service": {
      "id": "POOX2DQ",
      "type": "service_reference",
      "summary": "Default Service"
    },
    "created_at": "2026-11-17T23:36:31Z",
    "updated_at": "2026-11-17T23:53:48Z"
  }
}
```

#### `listAlerts`

```json
{ "more": false, "offset": 0, "total": null, "limit": 25, "alerts": [] }
```

#### `createAlert`

```json
{
  "message": "Event processed",
  "status": "success",
  "dedup_key": "87afb940a4904335a9b5f8764e33c40c"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Send a PagerDuty alert
```

### How it flows

1. **Manual Trigger:** Starts the workflow.
2. **PagerDuty Node:** Runs the selected operation.
3. **Log Node:** Displays the PagerDuty response.

A typical chain: **createIncident → acknowledgeIncident → resolveIncident**, passing `incident.id` with an expression.

### Example Configurations

#### Create an incident

```json
{
  "apiKey": "{{ secrets.PAGERDUTY_API_KEY }}",
  "fromEmail": "oncall@example.com",
  "operation": "createIncident",
  "serviceId": "POOX2DQ",
  "title": "Checkout API down",
  "urgency": "high",
  "body": "Health check failed 3 times in a row."
}
```

#### Send an alert

```json
{
  "apiKey": "{{ secrets.PAGERDUTY_API_KEY }}",
  "operation": "createAlert",
  "routingKey": "{{ secrets.PAGERDUTY_ROUTING_KEY }}",
  "title": "Disk usage above 90% on db-01",
  "severity": "warning"
}
```

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: security -->
## Security

> Store the PagerDuty `apiKey` and `routingKey` in Fusion's **Secrets** vault. Do not commit them into version control.

- A full-access API key can read and change every incident of the account: use a read-only key if the workflow only lists data.
- Anyone with the routing key can create alerts on the service: rotate it if it leaks.

<!-- /SECTION: security -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `incidentId is required` / `title is required` / `serviceId is required`
- **Cause:** A required parameter of the selected operation is empty.
- **Solution:** See the *Operations* table.

#### `routingKey is required (Events API V2 integration key of the service)`
- **Cause:** `createAlert` was run without a routing key.
- **Solution:** Add an Events API V2 integration to the service and copy its Integration Key.

#### `SyntaxError: Unexpected end of JSON input`
- **Cause:** PagerDuty answered with an empty body, usually a **401** (invalid key, or an account whose trial has expired). The node tries to read it as JSON.
- **Solution:** Check the key with `curl -i https://api.pagerduty.com/services -H "Authorization: Token token=<key>"`, and make sure the account is active.

#### `PagerDuty error (400)` mentioning the `From` header
- **Cause:** `fromEmail` is empty or is not a user of the account.
- **Solution:** Set `fromEmail` to the email of a PagerDuty user (`listUsers`).

#### `PagerDuty error (404): ... "Incident Not Found"`
- **Cause:** Wrong incident ID (for example the incident number instead of the ID).
- **Solution:** Use the ID from the incident URL or from `listIncidents`.

#### `PagerDuty error (400): ... "Incident Already Resolved"`
- **Cause:** The incident is already resolved.
- **Solution:** No action needed.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-11-17 | Full documentation: 9 operations, REST vs Events API, authentication, IDs, output examples, troubleshooting |
| 1.0.0 | 2026-08-04 | Initial documentation |

<!-- /SECTION: changelog -->
