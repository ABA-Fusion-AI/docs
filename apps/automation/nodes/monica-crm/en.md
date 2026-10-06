---
node_id: "monica-crm"
title: "Monica CRM"
description: "Monica personal CRM API. Manage contacts, activities, notes, and tasks."
category: "peer-only"
subcategory: "Integrations"
version: "1.0.0"
language: "en"
last_updated: "2026-10-06"
author: "Fusion Team"
tags:
  - monica
  - crm
  - contacts
  - personal-crm
  - integration
  - peer-only
related_nodes:
  - http-request
  - function
  - cron
---

<!-- SECTION: overview -->
# Monica CRM

> **Category:** Peer-Only Integrations &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Read and create data in [Monica](https://www.monicahq.com), an open-source personal CRM: list contacts, read a contact's notes and activities, list tasks, and create contacts. The node calls the Monica REST API with a personal access token and works with the hosted service (`app.monicahq.com`) and self-hosted Monica instances.

### Use Cases

- **Sync contacts:** Read your Monica contacts and send them to another tool or a report.
- **Add contacts automatically:** Create a contact when a form, webhook, or another CRM produces a new person.
- **Review interactions:** Read the notes and activities of a contact before a meeting.
- **Task reminders:** List tasks and send a reminder on a schedule.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `enum` | No | `getContacts` | `getContacts`, `createContact`, `getActivities`, `getNotes`, or `getTasks`. |
| `apiToken` | `string` | Yes | — | Monica personal access token, sent as `Authorization: Bearer <token>`. Supports expressions. |
| `baseUrl` | `string` | No | `https://app.monicahq.com/api` | API URL of your Monica instance. Set it only for a self-hosted Monica, e.g. `https://monica.example.com/api`. Supports expressions. |
| `contactId` | `string` | For `getNotes` | — | Contact ID (number). Required by `getNotes`; optional filter for `getActivities`. Supports expressions. |
| `requestBody` | `string` (JSON) | For `createContact` | — | JSON body of the new contact, sent as-is. See [Creating a Contact](#creating-a-contact). Supports expressions. |

### Available Operations

| Operation | Description | Monica request | Required |
|-----------|-------------|----------------|----------|
| `getContacts` | List contacts. | `GET /contacts` | — |
| `createContact` | Create a contact from `requestBody`. | `POST /contacts` | `requestBody` |
| `getActivities` | List activities, all or for one contact. | `GET /activities` or `GET /contacts/<contactId>/activities` | — |
| `getNotes` | List the notes of a contact. | `GET /contacts/<contactId>/notes` | `contactId` |
| `getTasks` | List tasks. | `GET /tasks` | — |

All parameters are shown for every operation. The list operations return the **first page** of results only.

### Creating a Contact

`requestBody` is sent to Monica without changes. Monica requires these fields — a body with only `first_name` is rejected with `422`:

```json
{
  "first_name": "Alice",
  "is_birthdate_known": false,
  "is_deceased": false,
  "is_deceased_date_known": false
}
```

Other fields such as `last_name` and `nickname` are optional.

### Getting an API Token

1. Sign in to Monica.
2. Open **Settings → API**.
3. Under **Personal Access Tokens**, click **Create New Token**, give it a name, and copy the token (it is shown only once).
4. Paste it into `apiToken`.

### Finding a Contact ID

The `id` of each contact is returned by `getContacts` (e.g. `817124`). It is also the number in the contact's URL in Monica.

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
| `success` | `object` | The Monica API response: `{ "data": ..., "links": ..., "meta": ... }` for lists, `{ "data": { ... } }` for `createContact`. |
| `error` | `Error` | `Monica CRM Error: <status> <statusText>` for API errors, or a validation message (`apiToken is required`, `contactId is required`, `requestBody is required`). |

### Output Examples

#### `getContacts`

```json
{
  "data": [
    {
      "id": 817124,
      "object": "contact",
      "first_name": "Alice",
      "last_name": null,
      "complete_name": "Alice",
      "gender": "Woman",
      "is_active": true
    }
  ],
  "links": {
    "first": "https://app.monicahq.com/api/contacts?page=1",
    "next": null
  },
  "meta": { "current_page": 1, "last_page": 1 }
}
```

#### `getNotes`

```json
{
  "data": [
    {
      "id": 91752,
      "object": "note",
      "body": "Met at the conference",
      "is_favorited": false,
      "contact": { "id": 817124 }
    }
  ],
  "meta": { "current_page": 1 }
}
```

#### `getActivities`

```json
{
  "data": [
    {
      "id": 44418,
      "object": "activity",
      "summary": "Lunch",
      "happened_at": "2026-10-06",
      "attendees": { "total": 1, "contacts": [{ "id": 817124 }] }
    }
  ],
  "meta": { "current_page": 1 }
}
```

#### `createContact`

```json
{
  "data": {
    "id": 817125,
    "object": "contact",
    "first_name": "Alice",
    "complete_name": "Alice"
  }
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Use Monica CRM in a workflow
```

### How it flows

1. **Manual Trigger:** Starts the workflow on demand.
2. **Monica CRM Node:** Runs the configured operation (e.g. `getContacts`).
3. **Log Node:** Displays the Monica response.

### Example Configurations

#### List the notes of a contact

```json
{
  "operation": "getNotes",
  "apiToken": "{{ secrets.MONICA_TOKEN }}",
  "contactId": "817124"
}
```

#### Create a contact from upstream data

```json
{
  "operation": "createContact",
  "apiToken": "{{ secrets.MONICA_TOKEN }}",
  "requestBody": "{\"first_name\":\"{{ input.firstName }}\",\"last_name\":\"{{ input.lastName }}\",\"is_birthdate_known\":false,\"is_deceased\":false,\"is_deceased_date_known\":false}"
}
```

#### Self-hosted Monica

```json
{
  "operation": "getContacts",
  "apiToken": "{{ secrets.MONICA_TOKEN }}",
  "baseUrl": "https://monica.example.com/api"
}
```

### Common Patterns

- **Webhook → Function → Monica CRM (`createContact`):** Build the contact JSON from a form submission and create it.
- **Cron → Monica CRM (`getTasks`) → Notification:** Send a daily list of open tasks.
- **Monica CRM (`getContacts`) → Function → Monica CRM (`getNotes`):** Loop over contacts and read their notes.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: security -->
## Security

> Store `apiToken` in Fusion's **Secrets** system. Never paste tokens directly into workflow parameters or commit them to version control.

- A personal access token gives full access to your Monica account — revoke it under **Settings → API** when it is no longer needed.
- For a self-hosted Monica, always use an HTTPS `baseUrl`.
- Personal CRM data (contacts, notes, activities) is sensitive; avoid logging full responses in shared workflows.

<!-- /SECTION: security -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Monica CRM Error: 401 Unauthorized`
- **Cause:** The token is wrong, revoked, or belongs to another Monica instance.
- **Solution:** Create a new token under **Settings → API** and check `baseUrl`.

#### `Monica CRM Error: 422 Unprocessable Entity` on `createContact`
- **Cause:** `requestBody` is not valid JSON or misses required fields. The node does not show Monica's detailed message.
- **Solution:** Send at least `first_name`, `is_birthdate_known`, `is_deceased`, and `is_deceased_date_known` (see [Creating a Contact](#creating-a-contact)).

#### `Monica CRM Error: 404 Not Found`
- **Cause:** `contactId` does not exist, or `baseUrl` is missing the `/api` suffix.
- **Solution:** Get IDs from `getContacts`; use a `baseUrl` ending in `/api`.

#### `contactId is required` / `requestBody is required`
- **Cause:** `getNotes` without `contactId`, or `createContact` without `requestBody`.
- **Solution:** Fill in the field required by the operation.

#### Only part of the records is returned
- **Cause:** List operations return the first page only (see `meta.last_page` and `links.next`).
- **Solution:** Use the HTTP Request node with `?page=<n>` for the next pages.

#### Hosted free plan limits
- **Cause:** The hosted Monica free plan is limited in the number of contacts.
- **Solution:** Upgrade the Monica plan or use a self-hosted instance.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-10-06 | Full documentation: parameters, operations, contact creation body, output examples, token setup, troubleshooting |
| 1.0.0 | 2026-08-04 | Initial documentation |

<!-- /SECTION: changelog -->