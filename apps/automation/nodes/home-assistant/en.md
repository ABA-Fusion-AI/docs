---
node_id: "home-assistant"
title: "Home Assistant"
description: "Control and query your Home Assistant smart home platform."
category: "peer-only"
subcategory: "integrations"
version: "1.0.0"
language: "en"
last_updated: "2026-09-25"
author: "Fusion Team"
tags: [integration, peer-only, home-assistant, smart-home]
related_nodes: []
---

<!-- SECTION: overview -->
# Home Assistant

> **Category:** Peer-only Integrations&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Connect to Home Assistant to read entity states, update a stored state, call services, retrieve history, and inspect configuration. The node authenticates with a long-lived access token and returns the parsed response from the selected operation.

### Use Cases

- Read a sensor or light state before branching a workflow.
- Call a service to control a device.
- Record a custom state with attributes.
- Retrieve entity history or available service definitions.
- Retrieve area IDs and instance configuration.

The node exposes nine operations. The current `listDevices` implementation does not enumerate the device registry; see its limitation below.
<!-- /SECTION: overview -->

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `operation` | Enum | No | `getStates` | One of the nine operations listed below. |
| `host` | String | Yes, at runtime | None | Instance base URL, for example `http://homeassistant.local:8123`. Supports expressions. |
| `token` | String | Yes, at runtime | None | Long-lived access token. Supports expressions. |
| `entityId` | String | For `getState`, `setState`, and `getHistory` | None | Entity ID, such as `light.kitchen`. Supports expressions. |
| `state` | String | For `setState` | None | Non-empty state value, such as `"on"` or `"23.5"`. Supports expressions. |
| `attributes` | String | No | None | JSON string sent as state attributes. Omitted or empty values become `{}`. Supports expressions. |
| `domain` | String | For `callService` | None | Service domain, such as `light` or `switch`. Supports expressions. |
| `service` | String | For `callService` | None | Service name, such as `turn_on`. Supports expressions. |
| `serviceData` | String | No | None | JSON string sent as the service request body. Omitted or empty values become `{}`. Supports expressions. |

Although `host` and `token` are optional in the schema, the handler rejects missing or empty values for every operation. Other required fields are checked only for the operation that uses them.

Use the instance base URL without an `/api` suffix. The node appends `/api` to build its request URLs. The instance must be reachable from the workflow runtime.

### Authentication

Every request includes:

- `Authorization: Bearer <token>`
- `Content-Type: application/json`

The node uses the configured token directly. It does not implement a login flow or token refresh.

### JSON Fields

Supply `attributes` and `serviceData` as strings containing JSON objects, rather than configuration objects. The handler parses non-empty strings with `JSON.parse`; invalid JSON throws before the HTTP request. It does not separately validate the parsed value's shape.

For `callService`, put the target entity inside `serviceData`. The separate `entityId` field is not added to the service payload.
<!-- /SECTION: configuration -->

<!-- SECTION: operations -->
## Operations

All endpoints below are relative to the configured host. Entity IDs, domains, and service names are URL-encoded where inserted into request paths or query parameters.

| Operation | Method | Endpoint | Required fields | Behavior |
| --- | --- | --- | --- | --- |
| `getStates` | GET | `/api/states` | Connection fields | Retrieve entity states. Default operation. |
| `getState` | GET | `/api/states/{entityId}` | `entityId` | Retrieve one entity's state. |
| `setState` | POST | `/api/states/{entityId}` | `entityId`, `state` | Send `{ state, attributes }`. |
| `listServices` | GET | `/api/services` | Connection fields | Retrieve available service definitions. |
| `callService` | POST | `/api/services/{domain}/{service}` | `domain`, `service` | Send the parsed `serviceData` as the request body. |
| `getHistory` | GET | `/api/history/period?filter_entity_id={entityId}&minimal_response=true` | `entityId` | Retrieve history using the endpoint's default time window and minimal responses. |
| `listAreas` | POST | `/api/template` | Connection fields | Render `{{ areas() | to_json }}` and parse the result as JSON. |
| `listDevices` | POST | `/api/template` | Connection fields | Render `{{ device_entities('') }}` and attempt to parse the result as JSON. |
| `getConfig` | GET | `/api/config` | Connection fields | Retrieve instance configuration. |

### State Updates and Device Control

`setState` changes the state representation in Home Assistant; it does not communicate with the physical device. To control a device, use `callService`, such as `light.turn_on`. See the [Home Assistant REST API](https://developers.home-assistant.io/docs/api/rest/).

The implementation always sends an `attributes` value with `setState`, using `{}` when the field is omitted. It does not fetch or merge existing attributes first.

### History

The request always includes `minimal_response=true`. The node exposes no start time, end time, or history options. It returns the server response without flattening or expanding history entries.

### Areas and Devices

`listAreas` explicitly serializes the template result with `to_json`.

`listDevices` uses the fixed expression `device_entities('')`. It supplies an empty identifier and has no configurable device ID or registry-listing request. Treat this as a limited implementation, not a reliable inventory of devices. An empty array may be returned, and a rendered value that is not valid JSON will cause a parsing error.

The template endpoint returns rendered text, while this node calls `res.json()` on every successful response. Therefore template output must itself be valid JSON. See the [template endpoint documentation](https://developers.home-assistant.io/docs/api/rest/#post-apitemplate).

### Request Limitations

Each execution sends one operation request after local validation. The handler implements no retries, custom timeout, pagination, or caching. It does not request service response data through a `return_response` option. Calling `stop()` does not cancel an in-progress request.
<!-- /SECTION: operations -->

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

- **Input:** An incoming event triggers the action. The handler does not read `incomingData`; request values come from configuration. Use expression-enabled fields for dynamic values.
- **Success:** The parsed JSON response is returned unchanged. No success wrapper or copy of the incoming event is added.
- **Error:** Validation, JSON parsing, network, and unsuccessful HTTP responses throw errors.

Typical response forms include state objects for `getState` and `setState`, an array of states for `getStates`, service definitions for `listServices`, history data for `getHistory`, area IDs for `listAreas`, and configuration data for `getConfig`. Exact fields depend on the server response. The `listDevices` caveat above applies.

Unsuccessful responses are read as text and included in the HTTP error. Successful responses are parsed as JSON, so an empty or non-JSON success body causes a parsing error.
<!-- /SECTION: inputs-outputs -->

<!-- SECTION: errors -->
## Errors & Troubleshooting

| Error or condition | What to check |
| --- | --- |
| `host and token are required` | Supply both non-empty connection fields. This check runs before operation-specific validation. |
| `entityId is required for getState` | Supply the entity ID to retrieve. |
| `entityId and state are required for setState` | Supply a non-empty entity ID and state string. |
| `domain and service are required for callService` | Supply both service path fields; include target information in `serviceData`. |
| `entityId is required for getHistory` | Supply the entity ID whose history should be retrieved. |
| `Home Assistant Error <status>: <response text>` | Inspect the server response, token, permissions, entity or service identifiers, and host URL. |
| JSON parsing error before a request | Check that `attributes` or `serviceData` is a valid JSON string for the selected operation. |
| JSON parsing error after a request | The successful response was not valid JSON. Check template output, especially for `listDevices`, and any proxy response. |
| Network error | Check runtime connectivity, hostname resolution, port, and TLS configuration. Fetch errors propagate directly. |
| `Unknown operation: <operation>` | An unsupported value reached the handler. The schema allows only the nine documented operations. |
<!-- /SECTION: errors -->

<!-- SECTION: examples -->
## Examples

Replace the host, token placeholder, and entity IDs with values for your instance.

### Read an Entity State

```json
{
  "operation": "getState",
  "host": "http://homeassistant.local:8123",
  "token": "<long-lived-access-token>",
  "entityId": "light.kitchen"
}
```

### Turn On a Light

```json
{
  "operation": "callService",
  "host": "http://homeassistant.local:8123",
  "token": "<long-lived-access-token>",
  "domain": "light",
  "service": "turn_on",
  "serviceData": "{\"entity_id\":\"light.kitchen\",\"brightness\":180}"
}
```

### Set a Custom Sensor State

```json
{
  "operation": "setState",
  "host": "http://homeassistant.local:8123",
  "token": "<long-lived-access-token>",
  "entityId": "sensor.workflow_temperature",
  "state": "23.5",
  "attributes": "{\"unit_of_measurement\":\"°C\",\"friendly_name\":\"Workflow Temperature\"}"
}
```

### Retrieve Entity History

```json
{
  "operation": "getHistory",
  "host": "http://homeassistant.local:8123",
  "token": "<long-lived-access-token>",
  "entityId": "sensor.workflow_temperature"
}
```

### List Area IDs

```json
{
  "operation": "listAreas",
  "host": "http://homeassistant.local:8123",
  "token": "<long-lived-access-token>"
}
```

For `getStates`, `listServices`, or `getConfig`, use the same connection fields and change `operation`. No operation-specific fields are required.

### Example Workflow

Connect a manual trigger to Home Assistant, select an operation, and configure its connection and required fields. Route the success output to a processing or logging step and the error output to an error-handling step.

```fusion-workflow
src: example.workflow.json
title: Use Home Assistant in a workflow
```
<!-- /SECTION: examples -->

<!-- SECTION: security -->
## Security

Store the access token securely and supply it through an expression-enabled field. Use HTTPS for remote connections. Keep tokens out of shared workflow exports and review server error text before sharing logs.
<!-- /SECTION: security -->

