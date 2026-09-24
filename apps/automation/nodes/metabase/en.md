---
node_id: "metabase"
title: "Metabase"
description: "Query data, manage cards and dashboards in Metabase."
category: "peer-only"
subcategory: "integrations"
version: "1.0.0"
language: "en"
last_updated: "2026-09-24"
author: "Fusion Team"
tags: [integration, peer-only]
related_nodes: []
---

<!-- SECTION: overview -->
# Metabase

> **Category:** Peer-only Integrations&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Connect to Metabase to run a saved card's query, retrieve a card or dashboard, or list cards, dashboards, and databases. The node authenticates with a username and password before each operation and returns the API's parsed JSON response.

### Use Cases

- Run an existing saved question and pass its results to another workflow step.
- Retrieve card or dashboard details for reporting workflows.
- List available cards, dashboards, or databases.

The implementation exposes six operations. It does not provide operations to create, update, or delete cards or dashboards, or to submit arbitrary SQL.
<!-- /SECTION: overview -->

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `host` | String | Yes | None | Metabase instance base URL, such as `https://metabase.example.com`. Supports expressions. |
| `username` | String | Yes | None | Username supplied to the Metabase session endpoint. Supports expressions. |
| `password` | String | Yes | None | Password supplied to the Metabase session endpoint. Supports expressions. |
| `operation` | Enum | No | `listCards` | `runQuery`, `getCard`, `listCards`, `listDashboards`, `getDashboard`, or `listDatabases`. |
| `cardId` | String | For `runQuery` and `getCard` | None | Saved card ID, for example `"42"`. Supports expressions. |
| `dashboardId` | String | For `getDashboard` | None | Dashboard ID, for example `"7"`. Supports expressions. |

Set `host` without a trailing slash or `/api` suffix; the node appends API paths directly and does not normalize the URL. The instance must be reachable from the workflow runtime.

`cardId` and `dashboardId` are optional in the schema but checked by the handler when their operation requires them. The handler rejects missing or empty IDs; it does not validate their format or URL-encode them. IDs are ignored by operations that do not use them.

### Authentication

For each operation that reaches the request handler, the node:

1. Sends `POST {host}/api/session` with a JSON body containing `username` and `password`.
2. Reads the session token from the response's `id` field.
3. Sends the operation request with the token in the `X-Metabase-Session` header.

Both requests use `Content-Type: application/json`. A successful execution makes two HTTP requests. The node does not cache or reuse the session token between executions, expose API-key authentication, or explicitly close the session afterward.
<!-- /SECTION: configuration -->

<!-- SECTION: operations -->
## Operations

All paths below are relative to `host`.

| Operation | Method | Endpoint | Required ID | Behavior |
| --- | --- | --- | --- | --- |
| `runQuery` | `POST` | `/api/card/{cardId}/query` | `cardId` | Execute the query for a saved card. |
| `getCard` | `GET` | `/api/card/{cardId}` | `cardId` | Retrieve a card's details. |
| `listCards` | `GET` | `/api/card` | None | Retrieve the card listing. This is the default operation. |
| `listDashboards` | `GET` | `/api/dashboard` | None | Retrieve the dashboard listing. |
| `getDashboard` | `GET` | `/api/dashboard/{dashboardId}` | `dashboardId` | Retrieve a dashboard's details. |
| `listDatabases` | `GET` | `/api/database` | None | Retrieve the database listing. |

`runQuery` sends no request body. The node does not expose query parameters, SQL text, or dashboard filters. Listing operations expose no pagination, filtering, or sorting options and return the response from a single API request.

The handler does not implement retries or a custom timeout. Calling `stop()` does not cancel requests already in progress.
<!-- /SECTION: operations -->

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

- **Input:** An incoming workflow event triggers the action. The handler does not read the incoming payload directly; request values come from configuration. Use expression-enabled fields for dynamic configuration values.
- **Success:** The parsed JSON response from the selected operation is returned unchanged. Its structure depends on the operation and the server response. The node adds no success wrapper, session token, or copy of the incoming data.
- **Error:** Missing required IDs, authentication failures, network failures, JSON parsing failures, and unsuccessful API responses throw errors.

Both authentication and operation responses are parsed as JSON before their HTTP status is checked. Empty or non-JSON responses therefore cause a parsing error before a custom HTTP error can be produced. There is no special handling for HTTP `204` responses.
<!-- /SECTION: inputs-outputs -->

<!-- SECTION: errors -->
## Errors & Troubleshooting

| Error or condition | What to check |
| --- | --- |
| `cardId is required` | Supply a non-empty `cardId` for `runQuery` or `getCard`. This check occurs before authentication. |
| `dashboardId is required` | Supply a non-empty `dashboardId` for `getDashboard`. This check occurs before authentication. |
| `Metabase auth error (<status>): <JSON response>` | The session request returned an unsuccessful HTTP status with a JSON body. Check the credentials and authentication response. |
| `Metabase: no session token returned` | The successful session response did not contain a truthy `id`. Check the configured host and session response. |
| `Metabase error (<status>): <JSON response>` | The operation returned an unsuccessful HTTP status with a JSON body. Check the resource ID, account permissions, and server response. |
| JSON parsing error | The authentication or operation endpoint returned an empty or non-JSON body. Check the host and any proxy or login-page response. |
| Network error | Check the host URL and connectivity from the workflow runtime. Fetch errors propagate without a custom wrapper. |
| `Unknown operation: <operation>` | An unsupported value reached the handler. The schema restricts selection to the six documented operations. |
<!-- /SECTION: errors -->

<!-- SECTION: examples -->
## Examples

Replace the connection placeholders and resource IDs with values for your instance. Keep real credentials out of shared examples.

### List Cards

```json
{
  "host": "https://metabase.example.com",
  "username": "<metabase-username>",
  "password": "<metabase-password>",
  "operation": "listCards"
}
```

### Run a Saved Card's Query

```json
{
  "host": "https://metabase.example.com",
  "username": "<metabase-username>",
  "password": "<metabase-password>",
  "operation": "runQuery",
  "cardId": "42"
}
```

To retrieve the card's details instead of executing its query, change `operation` to `getCard`.

### Retrieve a Dashboard

```json
{
  "host": "https://metabase.example.com",
  "username": "<metabase-username>",
  "password": "<metabase-password>",
  "operation": "getDashboard",
  "dashboardId": "7"
}
```

For a dashboard or database listing, use `listDashboards` or `listDatabases` with the same connection fields; neither operation requires an ID.

### Example Workflow

Connect a manual trigger to Metabase and route its success output to a Log or processing step. Configure the connection and selected operation before running it. Add an error-handling step for failed requests.

```fusion-workflow
src: example.workflow.json
title: Use Metabase in a workflow
```
<!-- /SECTION: examples -->

<!-- SECTION: security -->
## Security

Store credentials securely and supply them through expression-enabled fields. Use HTTPS for remote instances, and keep real passwords out of shared workflow exports. Error messages include serialized server responses, so review their contents before sharing logs.
<!-- /SECTION: security -->
