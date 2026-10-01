---
node_id: "netscaler"
title: "NetScaler ADC"
description: "Manage NetScaler ADC virtual servers, services, and service groups via NITRO API."
category: "peer-only"
subcategory: "integrations"
version: "1.0.0"
language: "en"
last_updated: "2026-10-01"
author: "Fusion Team"
tags: [integration, peer-only]
related_nodes: []
---

<!-- SECTION: overview -->
# NetScaler ADC

> **Category:** Peer-only Integrations&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Retrieve appliance statistics, list load-balancing virtual servers, services, and service groups, manage service bindings, and save configuration through the NITRO API. Each execution makes one request. The node shows its running status while executing.
<!-- /SECTION: overview -->

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `operation` | Enum | No | `getStats` | Select an operation from the table below. |
| `host` | String | Yes | — | Appliance base URL, such as `https://netscaler.example.com`. The node appends `/nitro/v1`. |
| `username` | String | Yes | — | Username sent in the `X-NITRO-USER` header. |
| `password` | String | Yes | — | Password sent in the `X-NITRO-PASS` header. |
| `vserverName` | String | No | — | Virtual server name for binding or unbinding. |
| `serviceName` | String | No | — | Service name for binding or unbinding. |
| `servicegroupName` | String | No | — | Sent as `servicegroupname` by `bindService`; ignored by other operations. |

All string parameters support expressions. The operation selector does not advertise expression support in the schema.

The node trims surrounding whitespace and trailing slashes from `host`. It rejects an empty normalized host or an empty username or password before sending a request. Supply the base URL without `/nitro/v1`.
<!-- /SECTION: configuration -->

<!-- SECTION: operations -->
## Operations

Paths below are appended to `{host}/nitro/v1`.

| Operation | Method | Path | Behavior |
| --- | --- | --- | --- |
| `getStats` | GET | `/stat/ns` | Retrieve appliance statistics. |
| `getVservers` | GET | `/config/lbvserver` | List load-balancing virtual servers. |
| `getServices` | GET | `/config/service` | List services. |
| `getServicegroups` | GET | `/config/servicegroup` | List service groups. |
| `bindService` | POST | `/config/lbvserver_service_binding` | Submit a binding using the configured names. |
| `unbindService` | DELETE | `/config/lbvserver_service_binding/{vserverName}` | Remove a binding, adding an `args=servicename:{serviceName}` query parameter when a nonempty service name is supplied. |
| `saveConfig` | POST | `/config/nsconfig?action=save` | Save configuration with the body `{"nsconfig":{}}`. |

### Bind a service

Select `bindService` and supply `vserverName` and `serviceName`. For virtual server `lb_web` and service `svc_web_01`, the request body is:

```json
{
  "lbvserver_service_binding": {
    "name": "lb_web",
    "servicename": "svc_web_01",
    "servicegroupname": ""
  }
}
```

All three properties are always sent; omitted names become empty strings. Binding names are optional in the schema and are not validated before the request. Supply the names needed for your binding; the appliance determines whether the request is valid.

`servicegroupName` is sent to this same service-binding endpoint. The implementation does not call a separate service-group binding endpoint.

### Unbind a service

Select `unbindService` and supply `vserverName` and `serviceName` to identify the binding. The virtual server name is URL-encoded, and the query string is encoded using `URLSearchParams`.

If `serviceName` is empty or omitted, the request has no `args` selector; the node does not guard against this case. `servicegroupName` is ignored, so this operation does not provide a service-group unbinding selector.

The listing operations do not use name parameters as filters. Binding operations do not automatically save configuration; run `saveConfig` separately to save changes.
<!-- /SECTION: operations -->

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

- **Input:** An incoming tick triggers execution. The handler does not read or merge the incoming payload; request values come from the node configuration.
- **Success:** The parsed API JSON is returned unchanged, without extracting resource arrays or wrapping the result. HTTP `204` returns `{"success":true}`.
- **Error:** The handler throws when configuration, transport, response reading, HTTP status, JSON parsing, or NITRO error checks fail.

Except for HTTP `204`, responses must contain valid JSON. A successful HTTP response is still rejected if its JSON object contains an `errorcode` other than the number `0` or the string `"0"`. A response without `errorcode` is accepted when the HTTP status is successful.
<!-- /SECTION: inputs-outputs -->

<!-- SECTION: examples -->
## Example Workflow

Connect **Manual Trigger → NetScaler ADC → Log**. Select `getVservers`, set your appliance base URL, and supply credential expressions for `username` and `password`. Connect the success output to Log to inspect the returned JSON.

```fusion-workflow
src: example.workflow.json
title: Inspect NetScaler ADC virtual servers
```

The supplied implementation uses `getVservers` to list virtual servers. If the accompanying exported example contains `listVirtualServers`, replace that value with `getVservers` when configuring the node. To retrieve statistics instead, select `getStats`; no name parameters are needed.
<!-- /SECTION: examples -->

<!-- SECTION: troubleshooting -->
## Troubleshooting

| Error or symptom | Meaning and next step |
| --- | --- |
| `NetScaler ADC host is required` | Set a nonempty appliance base URL. |
| `NetScaler ADC username and password are required` | Supply both credential values. |
| `NetScaler ADC request failed: check host, connectivity and TLS (redirects are refused)` | Check the host, network access, TLS, and whether the URL redirects. |
| `NetScaler ADC response read failed (HTTP …)` | A response arrived, but its body could not be read. |
| `authentication/authorization failed (HTTP …)` | The rejected or non-JSON response had status `401` or `403`, or its body matched a recognized authentication failure phrase. Check credentials and permissions. |
| `API error (HTTP …)` with a NITRO error code | The HTTP status was unsuccessful or `errorcode` was nonzero. Inspect the diagnostic excerpt and configured operation and names. |
| `non-JSON response` | Check that the host points to the appliance API rather than an HTML login page or proxy response. Empty responses other than HTTP `204` also fail JSON parsing. |

Response diagnostics collapse whitespace and include at most 300 characters of the response body after credential redaction. The implementation makes no retry and configures no explicit request timeout.
<!-- /SECTION: troubleshooting -->

<!-- SECTION: security -->
## Security

Use HTTPS and supply credentials through Fusion's credential system or secret expressions. Credentials are sent directly in NITRO headers; the node does not perform a separate login request or manage a session cookie. Redirects are refused.

Before including response excerpts in errors, the node replaces exact occurrences of the username and password, including their JSON-escaped, URL-encoded, and HTML-escaped forms, with `[REDACTED]`. Successful API responses are returned unchanged.

Remove credentials, variables, and node parameters from workflow exports before sharing or committing examples.
<!-- /SECTION: security -->
