---
node_id: "tailscale-routes"
title: "Tailscale Routes"
description: "Read and replace device subnet route approvals."
category: "actions"
subcategory: "integrations"
version: "1.0.0"
language: "en"
last_updated: "2026-11-16"
author: "Fusion Team"
tags: [tailscale, networking, vpn, infrastructure]
related_nodes: [tailscale-devices, tailscale-keys, tailscale-dns, http-request]
---

<!-- SECTION: header -->
# Tailscale Routes

> **Category:** Security & Networking → Network Tools | **Type:** Action Node

Automate tailnet administration through the Tailscale v2 REST API.
<!-- /SECTION: header -->

<!-- SECTION: overview -->
## Overview

This node manages Tailscale routes. Its operations are limited to this resource. It uses Axios from the shared `axios` package; no Tailscale daemon or SDK is required. It does not establish a VPN connection, run the Tailscale CLI, or provide webhook triggers.

### Authentication

Set `accessToken` to a Tailscale API access token or an already-issued OAuth access token, preferably through a Fusion secret expression. Device enrollment keys (`tskey-auth-...`) and OAuth client secrets are not API access tokens. This node does not exchange or refresh OAuth credentials.

Generate an API token from the Tailscale admin console's Keys page. Tokens expire; rotate the stored secret when needed. Scoped OAuth access tokens require permissions appropriate to the chosen operation: device read/write, route read/write, auth-key read/write, or DNS read/write. For OAuth-created auth keys, the tags must satisfy the OAuth client's tag ownership rules.
<!-- /SECTION: overview -->

<!-- SECTION: configuration -->
## Configuration

| Parameter | Required | Default | Description |
| --- | --- | --- | --- |
| `accessToken` | Yes | — | API or OAuth access token, sent using Bearer authentication. |
| `operation` | No | `getRoutes` | One of the operations below. |
| `deviceId` | For device/route operations | — | Device node ID (preferred) or legacy numeric ID as a string. |

Fields relevant to the selected operation appear in the configuration panel. Use expressions to reference incoming workflow data; input data does not automatically override configuration.

### Operations

Paths below are relative to `https://api.tailscale.com/api/v2`.

| Operation | Method and path | Additional parameters |
| --- | --- | --- |
| `getRoutes` | GET `/device/{deviceId}/routes` | — |
| `setRoutes` | POST `/device/{deviceId}/routes` | Required string array `routes` containing CIDRs |

`setRoutes` replaces the complete list of approved routes; it does not change routes advertised by the local daemon. Pass an explicit empty array to remove approvals. Read `getRoutes` first when preserving existing approvals.
<!-- /SECTION: configuration -->

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

An incoming tick executes one configured operation. Success outputs the unmodified JSON response, including its wrapper (`devices`, `keys`, and so on). Successful empty responses produce `{ "success": true, "statusCode": 200 }` (or the actual HTTP status, such as 204).

Errors use the normal node error output. Requests time out after 30 seconds. No automatic retry is performed, avoiding duplicate mutations. HTTP 401 indicates expired/invalid credentials; 403 indicates permissions; 404 indicates a missing resource; 429 indicates rate limiting. Axios headers and credentials are excluded from error messages.
<!-- /SECTION: inputs-outputs -->

<!-- SECTION: examples -->
## Example Workflow

Connect Manual Trigger → Tailscale Routes → Log. Set `accessToken` using a Fusion secret expression and choose `getRoutes`. Set `deviceId` to the target device node ID.

```fusion-workflow
src: example.workflow.json
title: Inspect device route approvals
```

The preview contains no credentials or parameters. Configure the node after adding it to a workflow.
<!-- /SECTION: examples -->

<!-- SECTION: related -->
## Related

- [Tailscale Devices](/docs/tailscale-devices)
- [Tailscale Keys](/docs/tailscale-keys)
- [Tailscale DNS](/docs/tailscale-dns)

- [HTTP Request](/docs/http-request) — Call Tailscale endpoints outside this node's supported operations.
- [Official API reference](https://tailscale.com/api)
- [API authentication](https://tailscale.com/docs/reference/tailscale-api)
- [Trust credentials and scopes](https://tailscale.com/docs/reference/trust-credentials)
- [Tailscale-maintained API client](https://github.com/tailscale/tailscale-client-go-v2) — Endpoint and payload cross-checks.
<!-- /SECTION: related -->
