---
node_id: "facebook-graph"
title: "Facebook Graph API"
description: "Send authenticated GET, POST, and DELETE requests to Facebook Graph API nodes and edges."
category: "security-networking"
subcategory: "apis-protocols"
version: "1.0.0"
language: "en"
last_updated: "2026-08-04"
author: "Fusion Team"
tags: [facebook, graph-api, meta, axios]
related_nodes: [http-request]
---

<!-- SECTION: overview -->
# Facebook Graph API

> **Category:** Security & Networking&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Run authenticated requests against Facebook's Graph API. The node supports versioned nodes and edges, custom fields and query parameters, and multipart binary uploads.
<!-- /SECTION: overview -->

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Default | Description |
|---|---|---:|---|---|
| `accessToken` | string | Yes | — | Facebook Graph API access token. |
| `hostUrl` | enum | No | `DEFAULT` | Use the standard or video Graph API host. |
| `httpRequestMethod` | enum | No | `GET` | HTTP method: GET, POST, or DELETE. |
| `graphApiVersion` | enum | No | `DEFAULT` | Graph API version, such as `v23.0`. |
| `node` | string | No | `me` | Graph node ID or alias. |
| `edge` | string | No | — | Optional edge such as `posts` or `insights`. |
| `ignoreSsl` | boolean | No | `false` | Disable certificate verification. Avoid in production. |
| `sendBinaryFile` | boolean | No | `false` | Send a multipart file with a POST request. |
| `inputBinaryFile` | string | No | `file:binary` | Form field and input path for the binary payload. |
| `fields` | array | No | — | Graph fields requested by GET operations. |
| `queryParameters` | array | No | — | Additional query name/value pairs. |
| `queryParametersJSON` | JSON | No | — | Additional query parameters as an object. |
<!-- /SECTION: configuration -->

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

- **Input:** Data used as the binary source when file upload is enabled.
- **Success:** Status code, normalized request details, and the Graph API response in `data`.
- **Error:** A normalized Facebook API error message.
<!-- /SECTION: inputs-outputs -->

<!-- SECTION: examples -->
## Example Workflow

```fusion-workflow
src: example.workflow.json
title: Fetch the authenticated Facebook profile
```
<!-- /SECTION: examples -->

<!-- SECTION: security -->
## Security

Store the access token as a workflow secret. Keep `ignoreSsl` disabled outside controlled development environments.
<!-- /SECTION: security -->
