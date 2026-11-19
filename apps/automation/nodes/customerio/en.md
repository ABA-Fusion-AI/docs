---
node_id: "customerio"
title: "Customer.io"
description: "Identify customers, track events, send transactional emails, read customers and list segments with the Customer.io Track and App APIs."
category: "peer-only"
subcategory: "Communication"
version: "1.0.0"
language: "en"
last_updated: "2026-11-19"
author: "Fusion Team"
tags:
  - customerio
  - customer-engagement
  - email
  - events
  - segments
  - crm
related_nodes:
  - mailgun
  - freshworks-crm
---

<!-- SECTION: overview -->
# Customer.io

> **Category:** Peer-Only Integrations &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Manage people and messaging in **Customer.io**: create or update **customers**, record **events**, send **transactional emails** and read **customers** and **segments**. Customer.io exposes two APIs with different hosts and credentials, and the node uses the right one for each of its **6 operations**:

| API | Auth | Operations |
|-----|------|------------|
| Track API | Site ID + Track API key (Basic) | `identifyCustomer`, `trackEvent`, `deleteCustomer` |
| App API | App API key (Bearer) | `sendEmail`, `getCustomer`, `listSegments` |

### Use Cases

- **User sync:** Create or update a customer in Customer.io when someone signs up in your app.
- **Behavior tracking:** Record events (purchase, login, cancellation) that trigger Customer.io campaigns.
- **Transactional email:** Send a password reset, receipt or welcome email from a workflow.
- **Audience checks:** Read a customer's attributes or list the workspace's segments.
- **Data removal:** Delete a customer on a GDPR request.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `region` | `enum` | Yes | Data center of the workspace: `us` (default) or `eu`. |
| `siteId` | `string` | For Track operations | Site ID of the workspace. |
| `apiKey` | `string` | For Track operations | Track API key (paired with the Site ID). |
| `appApiKey` | `string` | For App operations | App API key. |
| `operation` | `enum` | Yes | Operation to perform (default `identifyCustomer`). |
| `customerId` | `string` | Depends on operation | Customer identifier (`id`) in Customer.io. |
| `email` | `string` | No | Email of the customer (`identifyCustomer`), or recipient (`sendEmail`). |
| `eventName` | `string` | For `trackEvent` | Name of the event (e.g. `purchase`). |
| `eventData` | `string` | No | JSON object: event data for `trackEvent`, or `message_data` for `sendEmail` (e.g. `{"plan":"pro"}`). |
| `transactionalMessageId` | `string` | For `sendEmail` | ID (number) or trigger name of a transactional message. |

> All parameters are shown for every operation in the builder; only the ones listed in the table below are used.

### Operations

| Operation | Endpoint | Required | Optional |
|-----------|----------|----------|----------|
| `identifyCustomer` | `PUT {track}/customers/{customerId}` | `siteId`, `apiKey`, `customerId` | `email` |
| `trackEvent` | `POST {track}/customers/{customerId}/events` | `siteId`, `apiKey`, `customerId`, `eventName` | `eventData` |
| `deleteCustomer` | `DELETE {track}/customers/{customerId}` | `siteId`, `apiKey`, `customerId` | — |
| `sendEmail` | `POST {app}/send/email` | `appApiKey`, `transactionalMessageId`, `customerId` or `email` | `eventData` |
| `getCustomer` | `GET {app}/customers/{customerId}/attributes` | `appApiKey`, `customerId` | — |
| `listSegments` | `GET {app}/segments` | `appApiKey` | — |

| Region | Track host (`{track}`) | App host (`{app}`) |
|--------|------------------------|--------------------|
| `us` | `https://track.customer.io/api/v1` | `https://api.customer.io/v1` |
| `eu` | `https://track-eu.customer.io/api/v1` | `https://api-eu.customer.io/v1` |

> For `sendEmail`, a numeric `transactionalMessageId` is sent as a message ID; any other value is sent as a trigger name. When `customerId` is set it identifies the customer, otherwise `email` does; `email` is also used as the recipient address.

### Authentication & API Keys

1. Log in to **Customer.io** and open **Workspace Settings → API Credentials**.
2. **Track API key:** in the Track API keys list (legacy keys), copy the **Site ID** and its **API Key** (20 characters) into `siteId` and `apiKey`.
3. **App API key:** create or copy an **App API key** and paste it into `appApiKey`.
4. Choose the `region` of your workspace (`us` or `eu`). A wrong region returns `401`.

> The Track key and the App key are different: putting the App key in `apiKey` makes Track operations fail with `401 Unauthorized request`.

### Transactional message

To use `sendEmail`, create a message in **Transactional → Create message**, then copy its **ID** (shown on the message card, e.g. `2`) into `transactionalMessageId`.

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
| `success` | `object` | Response of Customer.io. When the API returns an empty body (identify, track, delete), the node returns `{ "success": true, "operation": "<operation>" }`. |
| `error` | `Error` | Emitted on missing fields, invalid JSON or HTTP errors (`Customer.io <operation> failed (HTTP <status>): ...`). Requests time out after 30 seconds. |

### Output Examples

#### `identifyCustomer` / `trackEvent` / `deleteCustomer`

```json
{ "success": true, "operation": "identifyCustomer" }
```

#### `getCustomer`

```json
{
  "customer": {
    "id": "fusion-test-1",
    "identifiers": {
      "cio_id": "c9a80e000203",
      "email": "user@example.com",
      "id": "fusion-test-1"
    },
    "attributes": {
      "cio_id": "c9a80e000203",
      "email": "user@example.com",
      "id": "fusion-test-1"
    },
    "timestamps": { "cio_id": 1795089684, "email": 0, "id": 1795089684 },
    "unsubscribed": false,
    "devices": []
  }
}
```

#### `listSegments`

```json
{
  "segments": [
    {
      "id": 1,
      "name": "All Users",
      "description": "Includes all the people in your workspace.",
      "state": "finished",
      "type": "dynamic",
      "tags": ["Sample"]
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
title: Use Customer.io in a workflow
```

### How it flows

1. **Manual Trigger:** Starts the workflow.
2. **Customer.io Node:** Runs the selected operation.
3. **Log Node:** Displays the response.

### Example Configurations

#### Identify a customer

```json
{
  "region": "us",
  "siteId": "{{ secrets.CIO_SITE_ID }}",
  "apiKey": "{{ secrets.CIO_TRACK_API_KEY }}",
  "operation": "identifyCustomer",
  "customerId": "{{ input.userId }}",
  "email": "{{ input.email }}"
}
```

#### Track an event

```json
{
  "region": "us",
  "siteId": "{{ secrets.CIO_SITE_ID }}",
  "apiKey": "{{ secrets.CIO_TRACK_API_KEY }}",
  "operation": "trackEvent",
  "customerId": "{{ input.userId }}",
  "eventName": "purchase",
  "eventData": "{\"amount\": 49.9, \"plan\": \"pro\"}"
}
```

#### Send a transactional email

```json
{
  "region": "us",
  "appApiKey": "{{ secrets.CIO_APP_API_KEY }}",
  "operation": "sendEmail",
  "transactionalMessageId": "2",
  "email": "{{ input.email }}",
  "eventData": "{\"firstName\": \"{{ input.firstName }}\"}"
}
```

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: security -->
## Security

> Store `siteId`, `apiKey` and `appApiKey` in Fusion's **Secrets** vault. Do not commit them into version control.

- The App API key has full read access to the workspace: give it only to workflows that need it.
- `deleteCustomer` permanently removes the person and their data: protect it from repeated triggers.
- Respect consent and unsubscribe rules when sending emails.

<!-- /SECTION: security -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Customer.io: "siteId" is required...` / `"apiKey" is required...` / `"appApiKey" is required...`
- **Cause:** The credentials of the API used by the operation are empty.
- **Solution:** Fill in Site ID + Track key for Track operations, or the App key for App operations.

#### `Customer.io: "customerId" is required...` / `"eventName" is required...` / `"transactionalMessageId" is required...`
- **Cause:** A required field is empty.
- **Solution:** Fill in the field listed in the error.

#### `Customer.io: "eventData" must be valid JSON` / `must be a JSON object`
- **Cause:** `eventData` is not a JSON object.
- **Solution:** Use an object such as `{"key": "value"}` (not an array or plain text).

#### `HTTP 401` with `Unauthorized request`
- **Cause:** Wrong key (often the App key in `apiKey`), wrong Site ID, or wrong region.
- **Solution:** Copy the Track key paired with the Site ID, and check the `region`.

#### `HTTP 404` on `getCustomer`
- **Cause:** No customer with this `customerId`.
- **Solution:** Create it with `identifyCustomer` first.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-11-19 | Full documentation: 6 operations, Track/App credentials, regions, output examples, troubleshooting |
| 1.0.0 | 2026-08-04 | Initial documentation |

<!-- /SECTION: changelog -->
