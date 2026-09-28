---
node_id: "pipedrive"
title: "Pipedrive"
description: "List deals, persons, and activities, or create deals and persons in Pipedrive."
category: "peer-only"
subcategory: "integrations"
version: "1.0.0"
language: "en"
last_updated: "2026-09-28"
author: "Fusion Team"
tags: [integration, pipedrive, crm, deals, contacts]
related_nodes: []
---

<!-- SECTION: overview -->
# Pipedrive

> **Category:** Peer-only Integrations | **Type:** Action Node

List deals, persons, or activities, and create deals or persons using the configured Pipedrive API token. Each execution sends one HTTP request and returns the parsed JSON response unchanged. The node displays its running status during execution.

Use it to retrieve CRM records for downstream processing, add a sales opportunity, or create a contact from values configured in the workflow.

This page describes the supplied implementation; it does not establish current API endpoint availability or server-side validation rules.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required by schema | Default | Description |
|-----------|------|--------------------|---------|-------------|
| `apiToken` | `string` | Yes | None | Token appended to the request URL as `api_token`. Supports expressions. The schema does not enforce a nonempty string. |
| `operation` | `enum` | No | `getDeals` | One of `getDeals`, `createDeal`, `getPersons`, `createPerson`, or `getActivities`. |
| `title` | `string` | No | Unset | Deal title sent by `createDeal`. Supports expressions. |
| `value` | `string` | No | Unset | Deal value sent by `createDeal`. Forwarded as a string without numeric conversion. Supports expressions. |
| `currency` | `string` | No | Unset | Currency sent by `createDeal`, for example `USD`. Supports expressions. |
| `name` | `string` | No | Unset | Person name sent by `createPerson`. Supports expressions. |
| `email` | `string` | No | Unset | Person email, placed in an array entry by `createPerson`. Supports expressions. |
| `phone` | `string` | No | Unset | Person phone, placed in an array entry by `createPerson`. Supports expressions. |

Creation fields are optional in the schema and have no operation-specific required checks in the handler. Supply the fields appropriate to the selected operation; the service determines whether the request is acceptable. The schema does not declare conditional visibility for these fields.

### Endpoint and authentication

The intended hardcoded base URL is:

```text
https://api.pipedrive.com/v1
```

**Source formatting caveat:** The pasted code contains a Markdown link (`[https://api.pipedrive.com/v1](https://api.pipedrive.com/v1)`) inside the `baseUrl` string. If this is present literally in the implementation, replace it with the plain URL above. Otherwise requests will not use a valid intended endpoint. There is no configurable `baseUrl` field.

Every request sends `Content-Type: application/json`. Authentication uses `?api_token=<apiToken>` in the URL, not an authorization header. The token is interpolated directly without URL encoding; avoid exposing request URLs containing credentials in logs or exported examples.

<!-- /SECTION: configuration -->

---

<!-- SECTION: operations -->
## Operations

Paths below are relative to the intended base URL. Each includes the `api_token` query parameter.

| Operation | Method | Path | Body |
|-----------|--------|------|------|
| `getDeals` | `GET` | `/deals` | None |
| `createDeal` | `POST` | `/deals` | Deal title, value, and currency |
| `getPersons` | `GET` | `/persons` | None |
| `createPerson` | `POST` | `/persons` | Person name, email array, and phone array |
| `getActivities` | `GET` | `/activities` | None |

### Create a deal

The request body is built with:

```javascript
JSON.stringify({ title, value, currency })
```

Undefined properties are omitted. Empty strings are retained. The node does not set owners, stages, organizations, or related persons, and does not forward arbitrary additional fields.

### Create a person

The request body is built with:

```javascript
JSON.stringify({
  name,
  email: [{ value: email }],
  phone: [{ value: phone }]
})
```

Email and phone arrays are always included. When `email` or `phone` is undefined, its array becomes `[{}]`, rather than being omitted. When supplied as an empty string, it becomes `[{ "value": "" }]`. The node does not validate email or phone formats locally.

### List records

List operations send no filters, sorting, or pagination parameters. Each execution returns only the response to that single request; the node does not fetch subsequent pages automatically. No record-ID lookup, update, or delete operation is exposed.

The schema restricts the operation to the five values above. If an unexpected value bypasses validation, the switch statement falls back to the deals GET request.

<!-- /SECTION: operations -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

| Port | Direction | Description |
|------|-----------|-------------|
| `input` | Input | Starts execution. The handler receives `incomingData` but does not read it. |
| `success` | Output | Parsed API JSON returned unchanged after a successful HTTP status. |
| `error` | Output | Error path for failed execution. |

Configure values directly or use expression-enabled fields to supply dynamic values through the workflow framework. Incoming objects are not automatically merged into configuration or forwarded as request bodies.

### Response handling

The node calls `res.json()` first, then checks `res.ok`. It returns the entire parsed JSON value, rather than extracting a `data` property, converting records, or adding an envelope. The actual response fields are determined by the service.

For a non-2xx response with valid JSON, it throws:

```text
Pipedrive error: <JSON-stringified response>
```

Network failures and JSON parsing failures propagate without this prefix. An empty or non-JSON response fails parsing even if its HTTP status indicates success. A successful HTTP response is returned without checking any application-level success flag inside the JSON.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

The token below is a placeholder. Supply your token through an appropriate private workflow configuration or expression.

### List deals

```json
{
  "apiToken": "<your-api-token>",
  "operation": "getDeals"
}
```

To list persons or activities, change `operation` to `getPersons` or `getActivities`. No other operation-specific fields are read.

### Create a deal

```json
{
  "apiToken": "<your-api-token>",
  "operation": "createDeal",
  "title": "Office equipment proposal",
  "value": "1500",
  "currency": "USD"
}
```

The POST body is:

```json
{
  "title": "Office equipment proposal",
  "value": "1500",
  "currency": "USD"
}
```

### Create a person

```json
{
  "apiToken": "<your-api-token>",
  "operation": "createPerson",
  "name": "Alex Example",
  "email": "alex@example.com",
  "phone": "+12025550123"
}
```

The POST body is:

```json
{
  "name": "Alex Example",
  "email": [{ "value": "alex@example.com" }],
  "phone": [{ "value": "+12025550123" }]
}
```

Each execution of a create operation sends a new POST request. The implementation supplies no deduplication or idempotency mechanism.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: List Pipedrive deals and inspect the response
```

Connect **Manual Trigger → Pipedrive → Log**. Set `apiToken` privately and select `getDeals`. Run the trigger and inspect the complete response in Log before mapping its fields in downstream nodes. Connect `error` to an error-handling branch if needed.

The example file contains only the wiring; configure the node before running it. Credentials and node parameters are omitted from the exported preview.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

| Symptom | Explanation and action |
|---------|------------------------|
| Invalid URL or request fails before reaching Pipedrive | Check that the source uses the plain base URL, without Markdown link syntax. |
| `Pipedrive error: ...` | The server returned a non-2xx response with valid JSON. Inspect the response details and verify token access and the selected operation's fields. |
| JSON parsing error | The response was empty or not valid JSON. Parsing occurs before the status check, so this error can hide the intended Pipedrive error message. |
| Input values appear ignored | `incomingData` is unused. Set configuration fields or expressions explicitly. |
| Deal value has the wrong type in configuration | The schema expects a string, such as `"1500"`, and forwards it without conversion. |
| Person request includes empty contact entries | Omitted email or phone becomes `[{}]`; inspect the generated payload and the server's validation response. |
| Fewer records than expected | The node makes a single list request and exposes no pagination controls. |
| Duplicate records after another run | Create operations submit a new request each time and do not check for existing records. |

The implementation defines no explicit timeout, automatic retry, or rate-limit handling. `stop()` is a no-op and does not abort an in-flight fetch.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-28 | Regenerated documentation from the supplied implementation, covering configuration, operations, payloads, response handling, and examples. |

<!-- /SECTION: changelog -->
