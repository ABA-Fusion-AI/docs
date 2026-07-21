---
node_id: "http-request"
title: "HTTP Request"
description: "Call any HTTP API and return the status, headers, and body, with configurable auth, query parameters, body encoding, response format, and timeout."
category: "utilities"
subcategory: "network"
version: "2.0.0"
language: "en"
last_updated: "2026-07-17"
author: "Fusion Team"
tags:
  - http
  - api
  - rest
  - request
  - webhook
  - integration
related_nodes:
  - function
  - webhook
  - gmail
---

<!-- SECTION: header -->

# HTTP Request

> **Category:** Utilities | **Type:** Action Node

Performs an HTTP request and returns the full response: `status`, `statusText`, `headers`, and `body`.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->

## Overview

The **HTTP Request** node calls any HTTP endpoint. It handles query-parameter
encoding, body serialization, authentication headers, and response decoding, and
hands the whole response back to the workflow.

### Key Features

- Every HTTP method: `GET`, `POST`, `PUT`, `PATCH`, `DELETE`, `HEAD`, `OPTIONS`
- Returns **status, headers and body** — not just the body
- Query parameters encoded for you, so no hand-written `%20`
- Body sent as JSON, form-urlencoded, or raw
- Basic and Bearer authentication without writing an `Authorization` header
- Response read as JSON, text, or **binary (base64)** — ready to feed a file field
- Per-request timeout, so a hung endpoint can't stall a worker forever
- Optionally treat `4xx`/`5xx` as a normal result and branch on the status

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->

## Configuration

### Parameters

| Parameter          | Type      | Required | Default | Description                                                                     |
| ------------------ | --------- | -------- | ------- | ------------------------------------------------------------------------------- |
| `url`              | `string`  | ✅ Yes   | —       | Full request URL                                                                 |
| `method`           | `enum`    | ❌ No    | `GET`   | `GET｜POST｜PUT｜PATCH｜DELETE｜HEAD｜OPTIONS`                                   |
| `queryParams`      | `record`  | ❌ No    | —       | Appended to the URL and percent-encoded automatically                            |
| `headers`          | `record`  | ❌ No    | —       | Request headers. Your `Content-Type` always wins                                 |
| `authType`         | `enum`    | ❌ No    | `none`  | `none｜basic｜bearer` — builds the `Authorization` header                        |
| `username`         | `string`  | ✅ Yes\* | —       | Basic auth username. Shown only when `authType=basic`                            |
| `password`         | `string`  | ✅ Yes\* | —       | Basic auth password. Shown only when `authType=basic`                            |
| `bearerToken`      | `string`  | ✅ Yes\* | —       | Sent as `Authorization: Bearer <token>`. Shown only when `authType=bearer`       |
| `bodyType`         | `enum`    | ❌ No    | `json`  | `json｜form-urlencoded｜raw`. Shown for `POST`/`PUT`/`PATCH`                     |
| `body`             | `any`     | ❌ No    | —       | Body object. Shown for `json` and `form-urlencoded`                              |
| `rawBody`          | `string`  | ❌ No    | —       | Sent verbatim. Shown for `bodyType=raw`                                          |
| `responseType`     | `enum`    | ❌ No    | `json`  | `json｜text｜binary` — how to read the response body                             |
| `timeoutMs`        | `number`  | ❌ No    | `30000` | Abort after this many ms. `0` disables the timeout                               |
| `ignoreHttpErrors` | `boolean` | ❌ No    | `false` | Resolve every status to `success` instead of erroring on `4xx`/`5xx`             |

\* Required only for the matching `authType`. Fields hidden by the current
selection are never required — the dependency rule suppresses the check — so
`username` is mandatory for basic auth without blocking bearer or none.

### Query parameters

Enter **raw values**; they are percent-encoded for you:

| Enter | Sent as |
|---|---|
| `Temperature Alert! Temp: 11.5` | `Temperature+Alert%21+Temp%3A+11.5` |

Do not pre-encode. If you paste an already-encoded value here it will be encoded
twice (`%20` becomes `%2520`). Values already written into the `url` are left
alone — encode those yourself, or move them here.

### Choosing a body format

`bodyType` decides both the serialization and the `Content-Type`:

| `bodyType` | `body` is | Sent as | Default `Content-Type` |
|---|---|---|---|
| `json` | an object | JSON | `application/json` |
| `form-urlencoded` | an object | `a=1&b=2`, url-encoded | `application/x-www-form-urlencoded` |
| `raw` | — (uses `rawBody`) | exactly what you typed | none — set it yourself |

**Setting a `Content-Type` header alone does not change the encoding.** The body
is serialized according to `bodyType`, so a form-urlencoded header with
`bodyType=json` still sends JSON. Pick `form-urlencoded` to actually form-encode
it. A `Content-Type` you set in `headers` is always preserved.

### Response format

| `responseType` | `body` contains |
|---|---|
| `json` | the parsed object |
| `text` | the raw string |
| `binary` | a **base64 string** of the bytes |

`binary` is how you move files through a workflow: bind `body` straight to a file
field such as a [Gmail](../gmail/en.md) attachment's `contentBase64`. The bytes
are preserved exactly — no UTF-8 decoding happens.

### Timeouts

`timeoutMs` defaults to 30s. Setting it to `0` disables the timeout entirely: a
server that accepts the connection and never answers will then hold an engine
slot until the workflow is stopped. Prefer a real number.

### Errors and status codes

By default any `4xx`/`5xx` sends the node to its **error** output, and the
response body is lost with it.

Set `ignoreHttpErrors` to `true` and every status resolves to the **success**
output instead, with the full envelope — so the graph can branch on `status` and
read the API's error payload:

```jsonc
// with ignoreHttpErrors: true
{
  "status": 404,
  "statusText": "Not Found",
  "headers": { "x-error-id": "abc123" },
  "body": { "error": "not found", "detail": "no such user" }
}
```

Network failures and timeouts still go to the error output — `ignoreHttpErrors`
only covers responses that actually arrived.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->

## Inputs & Outputs

### Outputs

| Output    | Description                                        |
| --------- | -------------------------------------------------- |
| `success` | The response envelope (below)                      |
| `error`   | Network failure, timeout, or a non-2xx status unless `ignoreHttpErrors` is on |

### Output shape

```jsonc
{
  "status": 200,
  "statusText": "OK",
  "headers": {
    "content-type": "application/json",
    "x-ratelimit-remaining": "99"
  },
  "body": { "id": 7, "ok": true }
}
```

Header names are lower-cased, as HTTP headers are case-insensitive.

> ### ⚠️ Breaking change in 2.0.0
>
> This node used to return **only the response body**. It now returns the
> envelope above, so response headers and the status code are reachable.
>
> **Any expression reading a field of the response must gain a `.body`:**
>
> | Before | After |
> |---|---|
> | `{{ $node.HTTP.id }}` | `{{ $node.HTTP.body.id }}` |
> | `{{ $node.HTTP }}` | `{{ $node.HTTP.body }}` |
>
> In exchange, this is now possible:
>
> ```jsonc
> {{ $node.HTTP.status }}                       // 200
> {{ $node.HTTP.headers["x-ratelimit-remaining"] }}
> ```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->

## Examples

### GET with query parameters

```json
{
  "url": "https://api.example.com/v1/users",
  "method": "GET",
  "queryParams": { "page": "2", "q": "ada lovelace" }
}
```

### POST JSON with a bearer token

```json
{
  "url": "https://api.example.com/v1/users",
  "method": "POST",
  "authType": "bearer",
  "bearerToken": "{{ $secrets.API_TOKEN }}",
  "bodyType": "json",
  "body": { "name": "Ada", "role": "admin" }
}
```

### Submit a form-urlencoded login

```json
{
  "url": "https://example.com/Account/LogOn",
  "method": "POST",
  "bodyType": "form-urlencoded",
  "body": { "username": "user", "password": "{{ $secrets.PASSWORD }}" }
}
```

### Download a file and email it

`responseType: "binary"` returns base64, which the Gmail node takes directly:

```json
{
  "url": "https://example.com/invoices/1042.pdf",
  "method": "GET",
  "responseType": "binary"
}
```

Then, in a downstream Gmail node:

```jsonc
{
  "operation": "send",
  "to": ["billing@example.com"],
  "attachments": [
    {
      "fileName": "invoice.pdf",
      "mimeType": "application/pdf",
      "contentBase64": "{{ $node.HTTP.body }}"
    }
  ]
}
```

### Branch on a 404 instead of failing

```json
{
  "url": "https://api.example.com/v1/users/999",
  "method": "GET",
  "ignoreHttpErrors": true
}
```

Follow it with an If/Else on `{{ $node.HTTP.status }}`.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Call an API and log the response
```

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->

## Troubleshooting

### My expressions broke after upgrading

The output is now `{ status, statusText, headers, body }`. Add `.body` — see the
breaking-change note above.

### The server rejects my form submission

Setting a `Content-Type: application/x-www-form-urlencoded` header is not enough
on its own — set **`bodyType` to `form-urlencoded`**, or the body is still
serialized as JSON.

### My query parameter arrives double-encoded

Values in `queryParams` are encoded for you. Enter `a b`, not `a%20b`.

### The request hangs forever

`timeoutMs: 0` disables the timeout. Set a real value; the node aborts and routes
to the error output.

### A 404 fails my workflow but I want to handle it

Set `ignoreHttpErrors` to `true` and branch on `status`.

### Binary downloads are corrupt

Set `responseType` to `binary`. Under `json` or `text` the bytes are decoded as
text and any byte that isn't valid UTF-8 is destroyed.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Function](./function.md) – Reshape a response body before using it
- [Gmail](../gmail/en.md) – Email a file downloaded with `responseType: binary`

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 2.0.0 | 2026-07-17 | **Breaking:** returns `{ status, statusText, headers, body }` instead of the bare body. Adds `PATCH`/`HEAD`/`OPTIONS`, query parameters, body format (json/form-urlencoded/raw), response format (json/text/binary), basic/bearer auth, `timeoutMs` (30s default), and `ignoreHttpErrors`. Form-urlencoded bodies are now actually form-encoded. Request headers and body are no longer written to worker logs. |
| 1.0.0 | 2026-03-11 | Initial release |

<!-- /SECTION: changelog -->
