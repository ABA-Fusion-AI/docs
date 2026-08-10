---
node_id: "header-validation"
title: "Header Validation"
description: "Validate HTTP headers — enforce no CRLF injection and maximum length limits."
category: "utilities"
subcategory: "Security & Validation"
version: "1.0.0"
language: "en"
last_updated: "2026-08-07"
author: "Fusion Team"
tags:
  - security
  - validation
  - headers
  - http
  - crlf
related_nodes:
  - http-request
  - webhook
  - function
---

<!-- SECTION: overview -->
# Header Validation

> **Category:** Utilities &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Validate HTTP header strings before they are used in outbound requests or forwarded through a pipeline. The node checks for **CRLF injection** (`\r\n`) and enforces a **maximum header length** — two common HTTP security requirements. Valid headers pass through unchanged; invalid ones emit an error.

### Use Cases

- **Security Hardening:** Sanitize headers received from user input or external sources before including them in HTTP requests, preventing CRLF injection attacks.
- **API Gateway Validation:** Validate headers at the entry point of a webhook-triggered workflow before forwarding them to internal services.
- **Header Compliance:** Enforce size limits on authorization tokens, session cookies, or custom headers to prevent oversized payloads.
- **Pre-Request Guard:** Place before an HTTP Request node to ensure outbound headers are safe and within spec.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `data` | `string` | No | — | The raw header string to validate. Each header should be on its own line in `Name: Value` format (e.g., `Content-Type: application/json\r\nAuthorization: Bearer token`). If omitted, reads from the incoming input. |
| `maxLength` | `number` | No | `100` | Maximum allowed length (in characters) for the entire header string. Headers exceeding this limit fail validation. |
| `disallowCRLF` | `boolean` | No | `true` | When enabled, the node rejects any header string containing `\r\n` (carriage return + line feed) sequences, preventing CRLF injection. |

### What is CRLF Injection?

CRLF injection is a security vulnerability where an attacker embeds `\r\n` characters into HTTP header values to inject additional headers or split the HTTP response. Example of a malicious input:

```
Authorization: Bearer token\r\nX-Injected: malicious-header
```

With `disallowCRLF: true`, the node catches this and emits an error before the header reaches any downstream HTTP call.

### Validation Rules

| Rule | Controlled by | Default |
|------|--------------|---------|
| No `\r\n` sequences allowed | `disallowCRLF` | `true` (enforced) |
| Total length ≤ N characters | `maxLength` | `100` |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `string` | The header string to validate. Used if `data` is not explicitly set. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `string` | The original header string, emitted unchanged if all validations pass. |
| `error` | `Error` | Emitted if the header contains a CRLF sequence or exceeds the `maxLength`. |

### Passthrough Behavior

The node does **not modify** the header string. If the input passes all validation checks, it is forwarded as-is on the `success` output. This allows it to be used as a safe guard in the middle of a pipeline without altering the data.

### Example — Valid Input

```
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiJ9
```

**Result:** Emitted on `success` unchanged.

### Example — Invalid Input (CRLF)

```
Authorization: Bearer token\r\nX-Injected: attack
```

**Result:** Emitted on `error` — `disallowCRLF` violation detected.

### Example — Invalid Input (Too Long)

```
X-Custom-Header: aaaaaaaaaa...aaaaaaaaaa  (> 100 characters)
```

**Result:** Emitted on `error` — `maxLength` exceeded.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Validate HTTP Headers Before Request
```

### How it flows

1. **Webhook / Manual Trigger:** Provides raw header data from an incoming request or user input.
2. **Header Validation Node:** Checks the header string for CRLF sequences and length violations.
3. **HTTP Request Node** (on `success`): Uses the validated header string to make a safe outbound call.
4. **Error Handler** (on `error`): Logs or rejects the request if the header is invalid.

### Common Patterns

- **Security Layer:** Place this node immediately after a Webhook trigger to sanitize any `Authorization`, `X-Api-Key`, or custom headers before they are used downstream.
- **Input Sanitization Pipeline:** Use in combination with a Function node to extract a specific header value from an incoming payload and validate it before forwarding.
- **Conditional Routing:** Connect the `error` output to a notification node (Slack, email) to alert on detected injection attempts.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Error emitted on a valid-looking header
- **Cause:** The `data` string contains hidden `\r\n` characters (e.g., copied from a raw HTTP dump or multiline string).
- **Solution:** Use a Function node before this node to strip or replace `\r\n` with `\n` if CRLF is expected in the input format but not for security purposes.

#### All headers fail `maxLength` validation
- **Cause:** The `maxLength` default of `100` is too short for long tokens such as JWT Bearer tokens, which can exceed 500 characters.
- **Solution:** Increase `maxLength` to a value appropriate for your headers (e.g., `1000` or `8192` for authorization tokens).

#### Node receives no data
- **Cause:** Both `data` and the upstream input are empty.
- **Solution:** Connect an upstream node that outputs the header string, or explicitly set `data` via expression.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [HTTP Request](./http-request.md) – Send an outbound request using the validated headers
- [Webhook](./webhook.md) – Receive and forward incoming HTTP headers for validation
- [Function](./function.md) – Pre-process or extract specific headers before validation

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-07 | Initial documentation |

<!-- /SECTION: changelog -->
