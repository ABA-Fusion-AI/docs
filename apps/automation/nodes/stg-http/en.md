---
node_id: "stg-http"
title: "STG HTTP Example"
description: "A small staging HTTP action used to demonstrate how a custom node defines parameters, sends a request, and returns a response."
category: "utilities"
subcategory: "stg"
version: "1.0.0"
language: "en"
last_updated: "2026-08-03"
author: "Fusion Team"
tags:
  - stg
  - http
  - api
  - example
  - testing
related_nodes:
  - http-request
  - stg-once
---

<!-- SECTION: header -->

# STG HTTP Example

> **Category:** Utilities | **Type:** Action Node

Sends a basic HTTP request and returns its status, headers, and body. This node
is intentionally named `stg-http` so it remains clearly separate from the
production `http-request` node.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->

## Overview

Use **STG HTTP Example** to verify staging deployments or as a small reference
when building a custom action node. It uses the Node.js built-in `fetch` API and
does not require an HTTP client dependency.

This example supports:

- `GET`, `POST`, `PUT`, `PATCH`, and `DELETE`
- Optional string-to-string request headers
- Raw string request bodies
- Automatic JSON encoding for object request bodies
- Automatic JSON parsing when the response declares a JSON content type

For authentication, query parameter helpers, timeouts, binary responses, and
more response controls, use the production `http-request` node instead.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->

## Configuration

| Parameter | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `url` | `string` | Yes | — | Full URL to request. Expressions are supported. |
| `method` | `enum` | No | `GET` | `GET`, `POST`, `PUT`, `PATCH`, or `DELETE`. |
| `headers` | `record<string, string>` | No | — | Optional request headers. Expressions are supported. |
| `body` | `unknown` | No | — | A string is sent unchanged; any other value is JSON-encoded. Ignored for `GET`. |

When the body is JSON-encoded, the node adds `Content-Type: application/json`
unless a content type was supplied in `headers`.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->

## Inputs & Outputs

### Inputs

| Input | Description |
| --- | --- |
| `input` | Starts one HTTP request. The example node does not otherwise transform the incoming value. |

### Outputs

| Output | Description |
| --- | --- |
| `success` | Emits the response envelope for a successful `2xx` response. |
| `error` | Emits an error for a network failure or non-`2xx` response. |

The success value has this shape:

```json
{
  "status": 200,
  "statusText": "OK",
  "headers": {
    "content-type": "application/json"
  },
  "body": {
    "message": "hello"
  }
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->

## Examples

### Basic GET

```json
{
  "url": "https://httpbin.org/get",
  "method": "GET"
}
```

### JSON POST

```json
{
  "url": "https://httpbin.org/post",
  "method": "POST",
  "headers": {
    "X-Example": "stg"
  },
  "body": {
    "message": "Hello from STG"
  }
}
```

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Call a test endpoint and inspect the response
```

<!-- /SECTION: examples -->
