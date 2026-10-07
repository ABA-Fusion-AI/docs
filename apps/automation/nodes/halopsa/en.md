---
node_id: "halopsa"
title: "HaloPSA"
description: "Manage tickets, clients, assets and users in HaloPSA"
category: "peer-only"
subcategory: "Integrations"
version: "1.0.0"
language: "en"
last_updated: "2026-10-07"
author: "Fusion Team"
tags:
  - halopsa
  - psa
  - helpdesk
  - tickets
  - itsm
  - integration
  - peer-only
related_nodes:
  - http-request
  - function
  - cron
---

<!-- SECTION: overview -->
# HaloPSA

> **Category:** Peer-Only Integrations &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Read and manage data in [HaloPSA](https://halopsa.com), a professional services automation and service desk platform: list and read tickets, create and update tickets, and list clients, assets and users. The node authenticates with an API application (Client ID and Secret, OAuth 2.0 client credentials) and calls the HaloPSA REST API of your instance.

### Use Cases

- **Create tickets automatically:** Open a ticket when a monitoring alert, form, or e-mail arrives.
- **Update tickets:** Change the summary, details, or priority of a ticket from another system.
- **Reporting:** Read tickets, clients, assets, or users and send them to a report or spreadsheet.
- **Sync data:** Copy HaloPSA clients or assets to another tool on a schedule.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `enum` | No | `getTickets` | `getTickets`, `getTicketById`, `createTicket`, `updateTicket`, `getClients`, `getClientById`, `getAssets`, or `getUsers`. |
| `baseUrl` | `string` | Yes | — | URL of your HaloPSA instance, without a trailing slash and without `/api`, e.g. `https://yourcompany.halopsa.com`. Supports expressions. |
| `clientId` | `string` | Yes | — | Client ID of the HaloPSA API application. Supports expressions. |
| `clientSecret` | `string` | Yes | — | Client Secret of the HaloPSA API application. Supports expressions. |
| `ticketId` | `string` | For `getTicketById`, `updateTicket` | — | Ticket ID (number). Supports expressions. |
| `clientId_param` | `string` | For `getClientById` | — | ID of the HaloPSA **client** (customer) to read. Not to be confused with the API `clientId` above. Supports expressions. |
| `summary` | `string` | No | — | Ticket summary (title), for `createTicket` and `updateTicket`. Supports expressions. |
| `details` | `string` | No | — | Ticket details (description), for `createTicket` and `updateTicket`. Supports expressions. |
| `ticketType` | `string` | No | — | Ticket type ID (number), for `createTicket` only. Supports expressions. |
| `priority` | `string` | No | — | Priority ID (number), for `createTicket` and `updateTicket`. Supports expressions. |
| `limit` | `number` | No | `50` | Maximum number of records returned by the list operations (sent as `count`). |

### Available Operations

| Operation | Description | HaloPSA request | Required |
|-----------|-------------|-----------------|----------|
| `getTickets` | List tickets. | `GET /api/Tickets?count=<limit>` | — |
| `getTicketById` | Read one ticket. | `GET /api/Tickets/<ticketId>` | `ticketId` |
| `createTicket` | Create a ticket. | `POST /api/Tickets` | — |
| `updateTicket` | Update a ticket's summary, details, or priority. | `POST /api/Tickets` with the ticket `id` | `ticketId` |
| `getClients` | List clients. | `GET /api/Client?count=<limit>` | — |
| `getClientById` | Read one client. | `GET /api/Client/<clientId_param>` | `clientId_param` |
| `getAssets` | List assets. | `GET /api/Asset?count=<limit>` | — |
| `getUsers` | List users (end-user contacts). | `GET /api/Users?count=<limit>` | — |

All parameters are shown for every operation. `ticketType` and `priority` take the numeric IDs configured in HaloPSA, not names (e.g. priority `3` = Medium in a default instance). Fields left empty in `createTicket` get HaloPSA's defaults (default ticket type, priority, and client).

### Authentication

Before each run, the node requests an access token from `<baseUrl>/auth/token` with `grant_type=client_credentials`, your `clientId` and `clientSecret`, and `scope=all`, then sends it as `Authorization: Bearer <token>` to `<baseUrl>/api/...`. The token is kept while the workflow runs and cleared when it stops.

### Creating the API Application

1. Sign in to HaloPSA as an administrator.
2. Open **Configuration → Integrations → HaloPSA API → View Applications** and click **New**.
3. Enter a name (e.g. `fusion`) and set **Authentication Method** to **Client ID and Secret (Services)**.
4. Set **Login Type** to **Agent** and select the agent the node acts as. HaloPSA applies both the application permissions and this agent's permissions.
5. On the **Permissions** tab, tick **all** (or only the areas you need).
6. Save, then copy the **Client ID** and **Client Secret** (the secret is shown only once) into the node.

### Finding IDs

- **Ticket ID:** the `id` returned by `getTickets` or `createTicket`, also shown as the ticket number in HaloPSA.
- **Client ID (`clientId_param`):** the `id` returned by `getClients`, or the `client_id` of a ticket.
- **Priority / ticket type IDs:** the `priority_id` and `tickettype_id` of existing tickets, or **Configuration → Tickets** in HaloPSA.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` | Triggers the node. The incoming data is not used directly; use expressions in the parameters to pass upstream values. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | The HaloPSA API response. List operations return `{ "record_count": <n>, "<items>": [ ... ] }` (`tickets`, `clients`, `assets`, or `users`); the other operations return the ticket or client object. |
| `error` | `Error` | `HaloPSA auth failed: <error>` when the token request fails, `HaloPSA API error (<status>): <response>` for API errors, or a validation message (`ticketId is required`, `clientId is required`). |

### Output Examples

#### `getTickets`

```json
{
  "record_count": 1,
  "tickets": [
    {
      "id": 2902,
      "dateoccurred": "2026-10-06T12:20:25.357",
      "summary": "Outlook Stuck",
      "status_id": 22,
      "tickettype_id": 1,
      "priority_id": 2,
      "client_id": 17,
      "client_name": "Acorn Construction",
      "site_name": "London HQ",
      "user_name": "Ben Castle",
      "team": "3rd Line Support"
    }
  ]
}
```

#### `createTicket` / `updateTicket` / `getTicketById`

```json
{
  "id": 2905,
  "dateoccurred": "2026-10-07T14:25:08.007",
  "summary": "Fusion test ticket (updated)",
  "details": "Created by the Fusion HaloPSA node test",
  "status_id": 1,
  "tickettype_id": 1,
  "sla_name": "Incident SLA",
  "priority_id": 1
}
```

#### `getClients` / `getClientById`

```json
{
  "record_count": 1,
  "clients": [
    { "id": 17, "name": "Acorn Construction", "inactive": false }
  ]
}
```

`getClientById` returns the client object itself (`{ "id": 17, "name": "Acorn Construction", ... }`).

#### `getAssets`

```json
{
  "record_count": 1,
  "assets": [
    {
      "id": 32,
      "inventory_number": "10CE010100D0G",
      "client_name": "Mario and Luigi's Pizza Place",
      "site_name": "New York",
      "assettype_name": "Mobile Phone",
      "inactive": false
    }
  ]
}
```

#### `getUsers`

```json
{
  "record_count": 1,
  "users": [
    {
      "id": 44,
      "name": "Ben Castle",
      "client_name": "Acorn Construction",
      "site_name": "London HQ",
      "emailaddress": "Ben.Castle@contoso.com",
      "inactive": false
    }
  ]
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Use HaloPSA in a workflow
```

### How it flows

1. **Manual Trigger:** Starts the workflow on demand.
2. **HaloPSA Node:** Gets a token and runs the configured operation (e.g. `getTickets`).
3. **Log Node:** Displays the HaloPSA response.

### Example Configurations

#### Create a ticket

```json
{
  "operation": "createTicket",
  "baseUrl": "https://yourcompany.halopsa.com",
  "clientId": "{{ secrets.HALOPSA_CLIENT_ID }}",
  "clientSecret": "{{ secrets.HALOPSA_CLIENT_SECRET }}",
  "summary": "Disk space low on {{ input.host }}",
  "details": "{{ input.message }}",
  "priority": "1"
}
```

#### Update a ticket

```json
{
  "operation": "updateTicket",
  "baseUrl": "https://yourcompany.halopsa.com",
  "clientId": "{{ secrets.HALOPSA_CLIENT_ID }}",
  "clientSecret": "{{ secrets.HALOPSA_CLIENT_SECRET }}",
  "ticketId": "2905",
  "summary": "Disk space low (resolved by cleanup)"
}
```

#### Read one client

```json
{
  "operation": "getClientById",
  "baseUrl": "https://yourcompany.halopsa.com",
  "clientId": "{{ secrets.HALOPSA_CLIENT_ID }}",
  "clientSecret": "{{ secrets.HALOPSA_CLIENT_SECRET }}",
  "clientId_param": "17"
}
```

### Common Patterns

- **Webhook → HaloPSA (`createTicket`):** Open a ticket from a monitoring alert or a form.
- **Cron → HaloPSA (`getTickets`) → Notification:** Send a daily summary of open tickets.
- **HaloPSA (`createTicket`) → HaloPSA (`updateTicket`):** Create a ticket, then update it with the `id` from the first node (`{{ input.id }}`).

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: security -->
## Security

> Store `clientId` and `clientSecret` in Fusion's **Secrets** system. Never paste them directly into workflow parameters or commit them to version control.

- The node acts as the agent selected in the API application; give that agent and the application only the permissions the workflow needs.
- Revoke or delete the API application in HaloPSA when it is no longer used.
- Always use an HTTPS `baseUrl`.
- Ticket, client, and user data can contain personal information; avoid logging full responses in shared workflows.

<!-- /SECTION: security -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `HaloPSA auth failed: invalid_client`
- **Cause:** Wrong `clientId` or `clientSecret`, or the application's authentication method is not **Client ID and Secret (Services)**.
- **Solution:** Check the application in **Configuration → Integrations → HaloPSA API**, or create a new secret.

#### `Unexpected token '<'` / `is not valid JSON`
- **Cause:** `baseUrl` is wrong (typo, `/api` added, or a trailing slash), so `/auth/token` returns an HTML page instead of a token.
- **Solution:** Use the plain instance URL, e.g. `https://yourcompany.halopsa.com`.

#### `HaloPSA API error (401)` or `(403)`
- **Cause:** The application or its agent lacks permission for the requested area, or the token expired in a long-running workflow.
- **Solution:** Tick the needed permissions on the application and the agent; restart the workflow to get a new token.

#### `HaloPSA API error (404)`
- **Cause:** `ticketId` or `clientId_param` does not exist.
- **Solution:** Get IDs from `getTickets` or `getClients`.

#### `ticketId is required` / `clientId is required`
- **Cause:** `getTicketById` or `updateTicket` without `ticketId`, or `getClientById` without `clientId_param`.
- **Solution:** Fill in the field required by the operation.

#### `updateTicket` does not change the ticket type
- **Cause:** `updateTicket` only sends `summary`, `details`, and `priority`; `ticketType` is used by `createTicket` only.
- **Solution:** Use the HTTP Request node to change other ticket fields.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-10-07 | Full documentation: parameters, operations, authentication, API application setup, output examples, troubleshooting |
| 1.0.0 | 2026-08-04 | Initial documentation |

<!-- /SECTION: changelog -->