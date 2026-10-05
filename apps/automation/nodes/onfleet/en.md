---
node_id: "onfleet"
title: "Onfleet"
description: "Delivery management with Onfleet"
category: "peer-only"
subcategory: "integrations"
version: "1.0.0"
language: "en"
last_updated: "2026-10-05"
author: "Fusion Team"
tags: [onfleet, delivery, tasks, integration, peer-only]
related_nodes: [http-request, function]
---

<!-- SECTION: overview -->
# Onfleet

> **Category:** Peer-only Integrations&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Use Onfleet to retrieve delivery tasks, create a task, list workers and teams, or retrieve organization information. Requests use the fixed base URL `https://onfleet.com/api/v2`.

### Use Cases

- Create delivery tasks from orders or workflow events.
- Retrieve tasks for reporting and downstream processing.
- Retrieve workers, teams, and organization information.

<!-- /SECTION: overview -->

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `apiKey` | `string` | Yes, in the schema | None | Onfleet API key. Supports expressions. |
| `operation` | `enum` | No | `getTasks` | `getTasks`, `createTask`, `getWorkers`, `getTeams`, or `getOrganization`. No expression metadata is declared. |
| `destination` | `string` | At runtime for `createTask` | None | Plain address or a string containing a destination JSON object. Supports expressions. |
| `recipientName` | `string` | No | None | Recipient name. Supports expressions. |
| `recipientPhone` | `string` | No | None | Recipient phone. Supports expressions. |
| `notes` | `string` | No | None | Task notes. Supports expressions. |

The four task fields are displayed only when `operation` is `createTask`. The handler does not perform a separate missing-key check for `apiKey`.

### Operations

| Operation | Method | Endpoint | Additional requirements |
|-----------|--------|----------|-------------------------|
| `getTasks` | GET | `/tasks` | None |
| `createTask` | POST | `/tasks` | Non-empty destination after trimming |
| `getWorkers` | GET | `/workers` | None |
| `getTeams` | GET | `/teams` | None |
| `getOrganization` | GET | `/organization` | None |

Every request sends `Content-Type: application/json` and `Authorization: Basic <base64(apiKey + ':')>`. The API key is the Basic authentication username, with an empty password.

### Task Body Construction

The node trims `destination`. A plain address becomes:

```json
{
  "destination": {
    "address": {
      "unparsed": "350 5th Avenue, New York, NY 10118, USA"
    }
  }
}
```

If the trimmed destination starts with `{` or `[`, the node parses it as JSON. It accepts a non-null object and sends that object directly as `destination`. Arrays are rejected. Destination object contents are not validated locally.

If either recipient field is truthy, the node adds one entry to `recipients`, with `name` and `phone`. A missing or empty companion field becomes an empty string. If both fields are empty or omitted, `recipients` is omitted. The node includes `notes` only when truthy. Recipient fields and notes are not trimmed.

<!-- /SECTION: configuration -->

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

- **Input:** Incoming workflow data (`unknown`) triggers execution but is not read directly by the handler. Parameters come from node configuration.
- **Success:** The parsed Onfleet JSON response is returned directly without wrapping or transformation.
- **Error:** Destination validation errors, HTTP errors, network failures, or JSON parsing failures.

The handler parses the response as JSON before checking `res.ok`. A non-success HTTP response containing valid JSON produces `Onfleet error: <JSON response>`. If JSON parsing fails, that parsing error propagates instead. The handler does not inspect API-level error fields in a successful HTTP response.

Each execution makes one request. The implementation supplies no pagination parameters, automatic page retrieval, retries, or custom timeout. The `stop()` method performs no cleanup or request cancellation.

<!-- /SECTION: inputs-outputs -->

<!-- SECTION: examples -->
## Examples

These objects represent node parameters. Replace the API key placeholder with a protected value or expression.

### Retrieve Tasks

Omitting `operation` selects `getTasks`.

```json
{
  "apiKey": "YOUR_ONFLEET_API_KEY"
}
```

### Create a Task from an Address

```json
{
  "apiKey": "YOUR_ONFLEET_API_KEY",
  "operation": "createTask",
  "destination": "350 5th Avenue, New York, NY 10118, USA",
  "recipientName": "Alex Smith",
  "recipientPhone": "+12125550123",
  "notes": "Leave the delivery at reception."
}
```

### Create a Task from Destination JSON

`destination` remains a string in node configuration, even when it contains JSON.

```json
{
  "apiKey": "YOUR_ONFLEET_API_KEY",
  "operation": "createTask",
  "destination": "{\"address\":{\"unparsed\":\"350 5th Avenue, New York, NY 10118, USA\"}}"
}
```

### Retrieve Workers

```json
{
  "apiKey": "YOUR_ONFLEET_API_KEY",
  "operation": "getWorkers"
}
```

### Retrieve Teams

```json
{
  "apiKey": "YOUR_ONFLEET_API_KEY",
  "operation": "getTeams"
}
```

### Retrieve Organization Information

```json
{
  "apiKey": "YOUR_ONFLEET_API_KEY",
  "operation": "getOrganization"
}
```

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Use Onfleet in a workflow
```

### Common Patterns

- Order event → Onfleet (`createTask`) → downstream task processing.
- Scheduled trigger → Onfleet (`getTasks`) → reporting.
- Manual trigger → Onfleet (`getWorkers` or `getTeams`) → downstream processing.

<!-- /SECTION: examples -->

<!-- SECTION: troubleshooting -->
## Troubleshooting

| Error or symptom | Cause | Resolution |
|------------------|-------|------------|
| `Destination address is required to create a task.` | Destination is missing, empty, or whitespace-only. | Supply a plain address or destination JSON object encoded as a string. |
| `Destination JSON is malformed. Enter a plain address or a valid destination JSON object.` | A destination starting with `{` or `[` cannot be parsed as JSON. | Correct the JSON syntax or use a plain address. |
| `Destination JSON must be a destination object.` | The parsed destination is an array. | Supply a destination object instead of an array. |
| `Onfleet error: <JSON response>` | Onfleet returned a non-success HTTP status with a JSON body. | Read the error details and check credentials and submitted fields. |
| JSON parsing failure | The response body is empty or not valid JSON. | Check the service response and connectivity. The handler expects JSON even for HTTP errors. |

The schema restricts `operation` to the five documented values. If an unsupported value nevertheless reaches the handler, the default branch requests `GET /tasks`.

<!-- /SECTION: troubleshooting -->

<!-- SECTION: security -->
## Security

Store credentials in Fusion's credential system. Do not place secrets directly in workflow parameters or exported examples.

Reference the API key through an expression and avoid exposing it in request logs. Requests use the fixed HTTPS Onfleet API URL; Basic authentication encodes the key in Base64.

<!-- /SECTION: security -->

<!-- SECTION: related -->
## Related

- [HTTP Request](../http-request/en.md) – Call additional endpoints or supply query parameters beyond those exposed by this node.
- [Function](../function/en.md) – Prepare destination strings, recipient information, and notes from workflow data.

<!-- /SECTION: related -->

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-10-05 | Regenerated documentation from the supplied implementation, including plain-address and destination JSON support. |

<!-- /SECTION: changelog -->
