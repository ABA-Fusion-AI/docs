---
node_id: "coap-client"
title: "CoAP Client"
description: "Send CoAP requests to CoAP servers using GET, POST, PUT, and DELETE methods over UDP."
category: "security-networking"
subcategory: "industrial-iot-protocols"
version: "1.0.0"
language: "en"
last_updated: "2026-08-06"
author: "Fusion Team"
tags:
  - coap
  - iot
  - udp
  - networking
  - protocols
  - api-client
related_nodes:
  - coap-server
  - http-request
  - mqtt
---

<!-- SECTION: header -->
# CoAP Client

> **Category:** Security & Networking | **Type:** Action Node

Send CoAP requests to compatible servers using the Constrained Application Protocol over UDP. The node supports GET, POST, PUT, and DELETE requests, configurable payloads, content formats, observation mode, confirmation settings, request timeouts, and automatic retries.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **CoAP Client** node sends requests to CoAP servers and returns a normalized response containing the response code, decoded payload, raw payload, response options, request settings, and execution timestamp.

CoAP is commonly used in constrained devices, IoT systems, sensors, embedded applications, and machine-to-machine communication.

The node accepts configuration values directly or allows supported values to be overridden by incoming workflow data.

### Key Features

- **Multiple Methods:** Supports GET, POST, PUT, and DELETE.
- **CoAP URL Support:** Accepts URLs with or without the `coap://` prefix.
- **Default CoAP Port:** Uses port `5683` when no port is provided.
- **Payload Support:** Sends string or JSON-serialized payloads.
- **Content Formats:** Supports common CoAP content-format values.
- **Observe Mode:** Supports CoAP observation requests.
- **Confirmable Messages:** Enables confirmable CoAP requests by default.
- **Timeout Handling:** Aborts requests that exceed the configured timeout.
- **Automatic Retries:** Retries failed requests with exponential delay.
- **Structured Output:** Returns response code, payload, headers, and request metadata.

### Supported Methods

| Method | Description | Payload |
|--------|-------------|---------|
| `GET` | Retrieve a CoAP resource. | Not normally required |
| `POST` | Submit data to a CoAP resource. | Supported |
| `PUT` | Create or replace a CoAP resource. | Supported |
| `DELETE` | Remove a CoAP resource. | Not normally required |

### Supported Content Types

| Content Type | CoAP Content-Format |
|--------------|---------------------|
| `text/plain` | `0` |
| `application/xml` | `41` |
| `application/octet-stream` | `42` |
| `application/json` | `50` |

> Unrecognized content types fall back to the CoAP content-format value used for `application/json`.

### Use Cases

- Query IoT devices
- Read sensor values
- Send commands to embedded devices
- Update constrained resources
- Test CoAP endpoints
- Communicate with machine-to-machine services
- Monitor observable CoAP resources
- Integrate CoAP services into automation workflows

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `url` | `string` | ✅ Yes | — | CoAP endpoint URL. The `coap://` prefix is added automatically when omitted. |
| `method` | `enum` | ❌ No | `GET` | Request method: GET, POST, PUT, or DELETE. |
| `payload` | `string` | ❌ No | — | Request payload. Displayed for POST and PUT methods. |
| `contentType` | `string` | ❌ No | `application/json` | Payload content type. Displayed for POST and PUT methods. |
| `observe` | `boolean` | ❌ No | `false` | Enables CoAP observation mode. |
| `confirmable` | `boolean` | ❌ No | `true` | Sends the request as a confirmable CoAP message. |
| `timeout` | `number` | ❌ No | `5000` | Request timeout in milliseconds. |
| `retries` | `number` | ❌ No | `2` | Number of retry attempts. Accepts values from `0` to `5`. |

### Default Values

| Parameter | Default |
|-----------|---------|
| `method` | `GET` |
| `contentType` | `application/json` |
| `observe` | `false` |
| `confirmable` | `true` |
| `timeout` | `5000` |
| `retries` | `2` |

### URL Handling

The node accepts a full CoAP URL:

```text
coap://coap.me/test
```

It also accepts a value without the protocol:

```text
coap.me/test
```

In that case, the node automatically adds:

```text
coap://
```

When no port is specified, the node uses:

```text
5683
```

### Payload Behavior

The `payload` field is primarily intended for POST and PUT requests.

When incoming workflow data contains a payload:

- string values are sent directly;
- non-string values are serialized using `JSON.stringify()`.

The payload is written to the CoAP request before the request is ended.

### Content-Type Behavior

When a payload is present, the node sets the CoAP `Content-Format` option.

Supported mappings:

```text
application/json         → 50
application/xml          → 41
text/plain               → 0
application/octet-stream → 42
```

### Retry Behavior

The node attempts the initial request plus the configured retry count.

For example:

```text
Retries: 2
```

allows up to:

```text
3 total attempts
```

The delay between retries increases exponentially.

### Configuration Override

Supported configuration values can be overridden by incoming workflow data:

```json
{
  "url": "coap://coap.me/test",
  "method": "GET",
  "observe": false,
  "confirmable": true,
  "timeout": 5000,
  "retries": 2
}
```

Incoming values take priority over the configured node values.

### Configuration Notes

- The `url` field is mandatory.
- `payload` and `contentType` are conditionally displayed for POST and PUT.
- `observe` defaults to `false`.
- `confirmable` defaults to `true`.
- The timeout is expressed in milliseconds.
- The maximum supported retry value is `5`.
- CoAP requests are sent over UDP.

<!-- /SECTION: configuration -->
---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` | Optional workflow input. Supported fields override the configured node parameters. |

The node accepts the following runtime overrides:

| Field | Type |
|------|------|
| `url` | string |
| `method` | string |
| `payload` | string \| object |
| `contentType` | string |
| `observe` | boolean |
| `confirmable` | boolean |
| `timeout` | number |
| `retries` | number |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `boolean` | Indicates whether the request completed successfully. |
| `url` | `string` | Final CoAP URL used for the request. |
| `method` | `string` | HTTP-like CoAP method used. |
| `code` | `string` | CoAP response code (for example `2.05`). |
| `payload` | `unknown` | Parsed as JSON when `contentType` is `application/json` and the response contains valid JSON; otherwise returned as a string. |
| `rawPayload` | `string` | Original response payload. |
| `contentType` | `string` | Request content type. |
| `observe` | `boolean` | Observation mode used during the request. |
| `confirmable` | `boolean` | Indicates whether the request was confirmable. |
| `headers` | `object` | CoAP response options returned by the server. |
| `timestamp` | `string` | ISO-8601 timestamp of the completed request. |

### Successful Response

Example:

```json
{
  "success": true,
  "url": "coap://coap.me/test",
  "method": "GET",
  "code": "2.05",
  "payload": "Welcome to the ETSI plugtest!",
  "rawPayload": "Welcome to the ETSI plugtest!",
  "contentType": "application/json",
  "observe": false,
  "confirmable": true,
  "headers": {
    "ETag": "...",
    "Content-Format": "text/plain"
  },
  "timestamp": "2026-08-06T20:42:04.208Z"
}
```

### Error Response

Examples:

```text
URL is required
```

```text
CoAP request timeout
```

```text
CoAP request failed after 3 attempts: <error message>
```

```text
CoAP request error: <error message>
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Basic Example: GET Request

Retrieve a resource from a CoAP server.

**Configuration**

```text
URL: coap://coap.me/test
Method: GET
```

---

### Example: POST Request

Send a JSON payload.

**Configuration**

```text
URL: coap://coap.me/test
Method: POST
Payload:
{
  "message": "Hello Fusion AI"
}
Content Type: application/json
```

---

### Example: PUT Request

Update an existing resource.

**Configuration**

```text
URL: coap://coap.me/test
Method: PUT
Payload:
Updated value
Content Type: text/plain
```

---

### Example: DELETE Request

Delete a resource.

**Configuration**

```text
URL: coap://coap.me/test
Method: DELETE
```

---

### Example: Observe Mode

Subscribe to an observable resource.

**Configuration**

```text
URL: coap://coap.me/obs
Method: GET
Observe: true
```

---

### Example: Custom Timeout

Increase the request timeout.

**Configuration**

```text
URL: coap://coap.me/test
Method: GET
Timeout: 10000
Retries: 3
```

---

### Example: Non-Confirmable Request

Send a non-confirmable CoAP message.

**Configuration**

```text
URL: coap://coap.me/test
Method: GET
Confirmable: false
```

---

### Example: Runtime Override

Incoming workflow data:

```json
{
  "url": "coap://coap.me/test",
  "method": "GET"
}
```

Runtime values override the node configuration.

---

### Example: Missing URL

If the URL is empty, the node throws:

```text
URL is required
```

<!-- /SECTION: examples -->
---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Send a CoAP Request
```

### Common Patterns

- **Read IoT Resource:** Manual Trigger → CoAP Client → Log
- **Send Sensor Command:** Manual Trigger → CoAP Client → Log
- **Update Device Configuration:** HTTP Request → CoAP Client → Log
- **Delete Remote Resource:** CoAP Client → Log
- **Observe Resource:** CoAP Client → Log

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### "URL is required"

**Cause**

The required `url` parameter is empty.

**Solution**

Provide a valid CoAP endpoint.

Example:

```text
coap://coap.me/test
```

---

#### "CoAP request timeout"

**Cause**

The server did not respond before the configured timeout.

**Solution**

- Increase the `timeout` value.
- Verify the server is reachable.
- Retry the request.

---

#### "CoAP request failed after X attempts"

**Cause**

The request failed after all retry attempts.

Possible reasons include:

- Server unavailable
- Network interruption
- Invalid hostname
- Invalid CoAP endpoint

**Solution**

Verify:

- URL
- Network connectivity
- Retry configuration
- Remote CoAP server availability

---

#### "CoAP request error"

**Cause**

The request failed while communicating with the CoAP server.

**Solution**

Inspect the underlying error message returned by the node.

---

#### Empty Payload

**Cause**

The server returned an empty response.

**Solution**

Verify that:

- the requested resource exists;
- the request method is correct;
- the server returns a payload for the requested endpoint.

---

#### Invalid JSON Payload

**Cause**

The response content is not valid JSON.

**Solution**

The node automatically returns the original response as a string when JSON parsing fails.

---

#### Observe Mode Does Not Produce Multiple Events

**Cause**

The remote server does not support CoAP Observe.

**Solution**

Verify that the server exposes an observable resource before enabling `observe`.

---

### Error Messages

| Error | Description |
|-------|-------------|
| `URL is required` | The required URL parameter is missing. |
| `CoAP request error: CoAP request failed after <attempts> attempts: CoAP request timeout` | The server did not respond before the timeout after all retry attempts. |
| `CoAP request error: CoAP request failed after <attempts> attempts: <error message>` | The initial request and all configured retries failed. |
| `CoAP request error: <error message>` | The node wrapped the underlying communication error. |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- HTTP Request
- MQTT
- CoAP Server

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-06 | Initial release of the CoAP Client documentation. |

<!-- /SECTION: changelog -->