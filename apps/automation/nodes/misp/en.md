---
node_id: "misp"
title: "MISP"
description: "Interact with MISP threat intelligence platform — events, attributes, taxonomies."
category: "peer-only"
subcategory: "integrations"
version: "1.0.0"
language: "en"
last_updated: "2026-10-02"
author: "Fusion Team"
tags: [integration, peer-only]
related_nodes: []
---

<!-- SECTION: overview -->
# MISP

> **Category:** Peer-only Integrations&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Interact with MISP threat intelligence platform — events, attributes, taxonomies.
<!-- /SECTION: overview -->

<!-- SECTION: configuration -->
## Configuration

Each execution performs one request using configured parameters. Set `host` to the MISP base URL without a trailing slash, for example `https://misp.example.com`. The node concatenates the host and operation path directly.

The API key is sent directly in the `Authorization` header without a `Bearer` prefix. Requests set `Content-Type: application/json` and `Accept: application/json`.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `host` | string | Yes | None | MISP base URL. Supports expressions. |
| `apiKey` | string | Yes | None | MISP API key. Supports expressions. |
| `operation` | enum | No | `getEvents` | Operation from the table below. |
| `eventId` | string | For `getEvent`, `addAttribute` | None | Event identifier. Supports expressions. |
| `info` | string | For `createEvent` | None | Event information. Supports expressions. |
| `distribution` | number | No | `0` | Event distribution value. |
| `threatLevel` | number | No | `1` | Sent as `threat_level_id`. |
| `analysis` | number | No | `0` | Event analysis value. |
| `attributeType` | string | For `addAttribute` | None | Sent as `type`. Supports expressions. |
| `attributeValue` | string | For `addAttribute` | None | Sent as `value`. Supports expressions. |
| `category` | string | No | `External analysis` at runtime | Attribute category. Supports expressions. An empty string does not trigger the fallback. |
| `searchValue` | string | No | None | Search value filter. Supports expressions. |
| `searchType` | string | No | None | Optional type filter; included only when nonempty. Supports expressions. |

Required operation-specific strings are checked at runtime and cannot be empty. Numeric parameters have no range restrictions in the supplied schema; MISP validates accepted values. Unrelated parameters are not sent.

### Operations

| Operation | Method | Path | Request details |
|-----------|--------|------|-----------------|
| `getEvents` | GET | `/events/index` | List events; no body or query parameters. |
| `getEvent` | GET | `/events/{eventId}` | Retrieve an event; requires `eventId`. |
| `createEvent` | POST | `/events` | Requires `info`; sends `info`, `distribution`, `threat_level_id`, and `analysis`. |
| `addAttribute` | POST | `/attributes/add/{eventId}` | Requires `eventId`, `attributeType`, and `attributeValue`; sends `type`, `value`, and `category`. |
| `searchEvents` | POST | `/events/restSearch` | Sends `value` and optional `type`. An omitted `searchValue` is omitted from serialized JSON. |
| `getTaxonomies` | GET | `/taxonomies` | Retrieve taxonomies; no body. |
| `getWarningLists` | GET | `/warninglists` | Retrieve warning lists; no body. |

The implementation provides no pagination controls, automatic retries, or explicit request timeout.
<!-- /SECTION: configuration -->

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

- **Input:** Triggers execution. The handler ignores incoming data and uses configured parameters. Use expression-enabled parameters for dynamic values.
- **Success:** The parsed JSON response is returned unchanged, without normalization or unwrapping. Its structure depends on the API response.
- **Error:** Validation, authentication, network, or remote-service failures are emitted through the error output.

The node calls `res.json()` before checking HTTP status. Unsuccessful responses containing JSON produce `MISP error (<status>): <JSON response>`. Empty or non-JSON responses cause a parsing failure, even with a successful HTTP status.
<!-- /SECTION: inputs-outputs -->

<!-- SECTION: examples -->
## Example Workflow

### Parameter Examples

Replace the host and API key placeholder with your connection settings. `YOUR_MISP_API_KEY` is a placeholder, not an expression.

```json
{
  "host": "https://misp.example.com",
  "apiKey": "YOUR_MISP_API_KEY",
  "operation": "createEvent",
  "info": "Indicators collected during an incident investigation",
  "distribution": 0,
  "threatLevel": 1,
  "analysis": 0
}
```

The request body uses `threat_level_id` instead of `threatLevel`.

```json
{
  "host": "https://misp.example.com",
  "apiKey": "YOUR_MISP_API_KEY",
  "operation": "addAttribute",
  "eventId": "123",
  "attributeType": "domain",
  "attributeValue": "indicator.example",
  "category": "External analysis"
}
```

The request body uses `type`, `value`, and `category`. Omitting `category` sends `External analysis`.

```json
{
  "host": "https://misp.example.com",
  "apiKey": "YOUR_MISP_API_KEY",
  "operation": "searchEvents",
  "searchValue": "indicator.example",
  "searchType": "domain"
}
```

This sends `{"value":"indicator.example","type":"domain"}`. Omit `searchType` to search without a type filter.

For listing events, retrieving taxonomies, or retrieving warning lists, use the same connection settings and select `getEvents`, `getTaxonomies`, or `getWarningLists`. For `getEvent`, also supply `eventId`.

### Workflow Integration

```fusion-workflow
src: example.workflow.json
title: Use MISP in a workflow
```

Configure the connection and operation before execution. Route `success` to downstream processing and `error` to error handling. Running status is enabled.
<!-- /SECTION: examples -->

<!-- SECTION: troubleshooting -->
## Troubleshooting

| Error or symptom | Action |
|------------------|--------|
| `eventId is required` | Supply a nonempty `eventId` for `getEvent` or `addAttribute`. |
| `info is required` | Supply nonempty `info` for `createEvent`. |
| `attributeType is required` | Supply a nonempty type for `addAttribute`. |
| `attributeValue is required` | Supply a nonempty value for `addAttribute`. |
| `MISP error (<status>): ...` | Inspect the response, API key permissions, host, and operation parameters. |
| JSON parsing failure | Check for empty responses or HTML from a proxy or authentication page. |
| Network failure | Check connectivity from the workflow runtime to the host. |
| Incoming values have no effect | Configure parameters explicitly or through expressions; the handler ignores incoming data. |

Unsupported operations are rejected by the enum schema. If one reaches the handler, it throws `Unknown operation: <operation>`.
<!-- /SECTION: troubleshooting -->

<!-- SECTION: security -->
## Security

Store credentials in Fusion's credential system. Do not place secrets directly in workflow parameters or exported examples.

Use HTTPS and resolve `apiKey` securely through an expression. Creating events and adding attributes modify your MISP instance. Responses and errors can contain threat intelligence data; control access to workflow outputs and logs.
<!-- /SECTION: security -->
