---
node_id: "validate-url"
title: "Validate URL"
description: "Validate URL structure, extract host and protocol components, and enforce strict protocol allowlists (HTTPS, WebSocket, SFTP, etc.)."
category: "data-transformation-etl"
subcategory: "validation-cleaning"
version: "1.0.0"
language: "en"
last_updated: "2026-09-01"
author: "Fusion Team"
tags:
  - validation
  - url
  - url-validator
  - security
  - protocols
  - https
  - domain
  - host
  - data-cleaning
  - data-transformation
related_nodes:
  - schema-validate
  - validate-phone
  - html-sanitize
  - http-request
  - webhook
  - function
---

<!-- SECTION: header -->
# Validate URL

> **Category:** Data Transformation (ETL) | **Subcategory:** Validation & Cleaning | **Type:** Action Node

Validate incoming web addresses against official URL standards, verify format integrity, extract structured components (`host`, `protocol`, clean `url`), and enforce protocol allowlists (e.g. restrict strictly to `https` or `wss`).

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Validate URL** node acts as an automated security and format gatekeeper for web addresses within your workflows. Before making an HTTP request, registering a webhook callback, storing a website in a database, or opening a real-time socket connection, this node ensures that the URL is syntactically valid and complies with your permitted protocols.

When a URL passes validation, the node breaks it down into structured fields (`valid: true`, `url`, `protocol`, `host`), making it easy to route data based on domain or protocol without writing complex regular expressions.

```
┌────────────────────────┐
│    Trigger / Webhook   │  (Inbound user payload with website URL)
└───────────┬────────────┘
            │
            ▼
┌────────────────────────┐
│      Validate URL      │  (Allowed: ["https"])
└───────────┬────────────┘
            │
      ┌─────┴─────┐
      │           │
  [success]    [error]
      │           │
      ▼           ▼
┌───────────┐ ┌───────────┐
│ Downstream│ │   Error   │  (Reject invalid / insecure link)
│   Nodes   │ │  Handler  │
└───────────┘ └───────────┘
```

### What You Can Do With This Node

- **Verify URL Integrity:** Check if an incoming string is a well-formed URL according to standard RFC 3986 specifications.
- **Enforce Protocol Allowlists:** Restrict acceptable URLs strictly to safe protocols (e.g. `["https"]` to block insecure `http`, or `["ws", "wss"]` for real-time streaming sockets).
- **Extract URL Components:** Automatically isolate the `host` (domain with optional port) and `protocol` for downstream routing and domain filtering.
- **Clean & Normalize:** Automatically trim surrounding whitespace and extract URL values from strings or structured JSON payloads.
- **Graceful Error Branching:** Route invalid or disallowed URLs to the red `error` output to notify users, reject form submissions, or trigger alerts.

---

### Common Use Cases

- **Webhook Callback Registration:** Verify that customer-provided callback URLs use secure `https://` before saving them to your API backend.
- **Lead & Form Validation:** Screen website addresses submitted through lead generation forms and reject incomplete links (such as `google.com` missing `https://`).
- **Domain Allowlisting & Routing:** Extract the `host` property to route requests to specific regional servers or block untrusted third-party domains.
- **Multi-Protocol Gateways:** Validate and route different protocols (HTTP, HTTPS, WebSocket, SFTP) to their respective specialized connector nodes.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

Add the **Validate URL** node to your workflow canvas and click it to open the configuration modal.

![Validate URL Configuration](icon.svg)

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|:----:|:--------:|:-------:|-------------|
| `Data` | `string` | ❌ No | — | The URL string to validate (e.g. `"https://example.com/api/v1"`). If left empty, the node validates the incoming payload from the `input` connection. |
| `AllowedProtocols` | `array` | ❌ No | `[]` | List of allowed protocols without colons (e.g. `["https"]`, `["http", "https"]`, `["ws", "wss"]`). If empty, any valid standard protocol is accepted. |

---

### Parameter Details & How to Configure

#### 1. `Data` (Optional)
The target web address you want to validate.
- **Static Entry:** Enter a complete URL directly (e.g. `https://example.com/login`).
- **Dynamic Expression:** Click **Expression** and enter `outputs.Webhook.success.body.website` or `outputs.FormTrigger.success.url` to dynamically validate data from upstream nodes.
- **Fallback:** If left empty, the node automatically reads the incoming data payload from the upstream connected node.

> [!IMPORTANT]
> **Complete URL Required:** A valid URL must include a protocol scheme (e.g. `https://www.google.com`). Strings like `www.google.com` without a scheme will fail validation because they lack a defined network protocol.

#### 2. `AllowedProtocols` (Optional)
Specify an allowlist of permitted protocol schemes.
- **How to Add:** Click **+ Add Item** and type the protocol name **without** a colon (e.g. `https`, `http`, `wss`, `ws`, `ftp`, `sftp`).
- **Empty List (`[]`):** Accepts any syntactically valid protocol scheme recognized by standard URL parsers.
- **Security Recommendation:** For production webhooks and API integrations, add `https` to prevent accidental insecure HTTP transmissions.

---

### Switching Between Text and Expression Mode

You can toggle between direct text entry and dynamic expressions for both the `Data` parameter and individual items in `AllowedProtocols`:

```
┌─────────────────────────────────────────────────────────────┐
│ Parameters                                                  │
│                                                             │
│ Data (Optional)                                 [Expression]│
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ outputs.Webhook.success.body.callbackUrl                │ │
│ └─────────────────────────────────────────────────────────┘ │
│                                                             │
│ AllowedProtocols (Optional)                     [Expression]│
│                                + Add Item                   │
│ 0                                               [Expression]│
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ https                                                   │ │
│ └─────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

- **Standard Mode:** Type fixed strings directly into the inputs.
- **Expression Mode:** Click **Expression** to bind dynamic values from previous nodes using `outputs.<NodeLabel>.<outputPort>.<field>`.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Input Port

| Port | Description |
|------|-------------|
| `input` | Receives data from upstream triggers or action nodes. If `Data` is not defined in the node configuration, this input payload is validated as the URL. |

### Output Ports

| Port | Color | Description |
|------|:-----:|-------------|
| `success` | 🟢 Green | Emitted when the URL is valid and satisfies the protocol allowlist. Returns structured components (`url`, `protocol`, `host`). |
| `error` | 🔴 Red | Emitted when the URL format is invalid, the protocol is not allowed, or the data is empty. |

---

### What the Output Looks Like

When validating `https://example.com/api/v1/users?page=1&limit=20` with `AllowedProtocols: ["https"]`:

```json
{
  "valid": true,
  "url": "https://example.com/api/v1/users?page=1&limit=20",
  "protocol": "https",
  "host": "example.com"
}
```

When validating a URL with a custom port: `https://api.subdomain.example.co.uk:8080/v2/items#section-top`:

```json
{
  "valid": true,
  "url": "https://api.subdomain.example.co.uk:8080/v2/items#section-top",
  "protocol": "https",
  "host": "api.subdomain.example.co.uk:8080"
}
```

---

### Output Field Reference

| Field | Type | Description | Example |
|-------|:----:|-------------|---------|
| `valid` | `boolean` | Confirms that validation passed successfully. Always `true` on the `success` output. | `true` |
| `url` | `string` | The complete, cleaned, and trimmed URL string. | `"https://example.com/api/v1/users?page=1&limit=20"` |
| `protocol` | `string` | The extracted protocol scheme (lowercase, without colon). | `"https"`, `"wss"`, `"ftp"` |
| `host` | `string` | The domain hostname (including port number if specified). | `"example.com"`, `"api.example.co.uk:8080"` |

---

### How to Use Output Data in Downstream Nodes

Use these expressions to reference validated URL components in subsequent workflow steps:

| Desired Data | Expression Syntax | Typical Next Step |
|--------------|-------------------|-------------------|
| **Clean Validated URL** | `outputs.ValidateUrl.success.url` | Pass to [HTTP Request](../http-request/en.md) node or save to database |
| **Domain / Host** | `outputs.ValidateUrl.success.host` | Check against domain allowlists or route to specific APIs |
| **Protocol Scheme** | `outputs.ValidateUrl.success.protocol` | Branch workflow between HTTPS, WebSockets, or FTP connections |
| **Validation Status** | `outputs.ValidateUrl.success.valid` | Conditional checks in branching logic |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Step-by-Step Usage Examples

### Example 1: Enforcing Strict HTTPS for Webhook Callbacks

Ensure user-submitted webhook callback endpoints are encrypted with HTTPS:

**Configuration:**
- **Data:** `outputs.Webhook.success.body.targetUrl`
- **AllowedProtocols:** `["https"]`

**Outcomes:**
- ✅ `https://my-app.com/events` ➔ Emitted on `success` port (`protocol: "https"`, `host: "my-app.com"`).
- ❌ `http://insecure-site.com/events` ➔ Emitted on `error` port (`Protocol 'http' is not allowed. Allowed: https`).

---

### Example 2: Validating WebSocket Streaming Feeds

Allow only real-time WebSocket stream URLs:

**Configuration:**
- **Data:** `wss://stream.binance.com/ws`
- **AllowedProtocols:** `["ws", "wss"]`

**Output (`success`):**
```json
{
  "valid": true,
  "url": "wss://stream.binance.com/ws",
  "protocol": "wss",
  "host": "stream.binance.com"
}
```

---

### Example 3: Verifying Complex URLs with Custom Ports and Hash Fragments

Validate endpoints containing subdomain hierarchies, custom port numbers, and URL hash anchors:

**Configuration:**
- **Data:** `https://api.subdomain.example.co.uk:8080/v2/items#section-top`
- **AllowedProtocols:** `[]` *(Accept any valid protocol)*

**Output (`success`):**
```json
{
  "valid": true,
  "url": "https://api.subdomain.example.co.uk:8080/v2/items#section-top",
  "protocol": "https",
  "host": "api.subdomain.example.co.uk:8080"
}
```

---

### Example 4: Screening Form Submissions with Automatic Prefixing

If user input might lack a scheme (e.g. `"google.com"`), use an upstream **Function** node to add `https://` before validation:

```javascript
// Prepend https:// if no protocol exists
let rawUrl = input.website.trim();
if (!/^https?:\/\//i.test(rawUrl)) {
  rawUrl = "https://" + rawUrl;
}

return {
  sanitizedUrl: rawUrl
};
```

Then in **Validate URL**, set **Data** to `outputs.Function.success.sanitizedUrl`.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Validate URL format and enforce HTTPS protocol allowlist
```

### Common Workflow Architecture Patterns

#### Pattern 1: Secure Inbound Webhook Dispatcher
```text
[Webhook: Register Subscriber]
          ↓
[Validate URL] (Data: outputs.Webhook.success.body.endpoint, Allowed: ["https"])
    ├── (success) ──> [Database: Save Subscriber]
    └── (error)   ──> [Webhook Response: Return 400 Bad URL]
```

#### Pattern 2: Multi-Protocol Network Router
```text
[Inbound Feed Event]
          ↓
[Validate URL] (Allowed: ["https", "wss", "sftp"])
          ↓
[Branch / Switch: By outputs.ValidateUrl.success.protocol]
    ├── (https) ──> [HTTP Request Node]
    ├── (wss)   ──> [WebSocket Connect Node]
    └── (sftp)  ──> [SFTP File Upload Node]
```

#### Pattern 3: Lead Enrichment & Domain Classification
```text
[Form Trigger: Company Signup]
          ↓
[Validate URL] (Data: outputs.FormTrigger.success.companyWebsite)
          ↓
[Function / Clearbit API] (Query Company Info using outputs.ValidateUrl.success.host)
```

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Problems & Solutions

#### 1. `URL validation failed: Invalid URL format: www.google.com`
- **Why it happens:** The input string does not contain a protocol scheme (like `https://` or `http://`).
- **How to fix:** Ensure all URLs start with a valid scheme. If accepting freeform user input, use a **Function** node to prepend `https://` to naked domain names before validation.

#### 2. `URL validation failed: Protocol 'http' is not allowed. Allowed: https`
- **Why it happens:** The URL uses `http://`, but `AllowedProtocols` is configured to only permit `https`.
- **How to fix:** Either update the source data to use `https://`, or add `"http"` to the `AllowedProtocols` list if unencrypted communication is acceptable for your use case.

#### 3. `Data is required for URL validation` or `Cannot validate empty URL`
- **Why it happens:** The `Data` parameter was left blank, and no data was received from the upstream node connection.
- **How to fix:** Verify that the upstream node is producing output and that your expression (e.g. `outputs.Webhook.success.body.url`) points to a populated string.

---

### Error Quick Reference Table

| Error Message | Cause | Recommended Action |
|---------------|-------|--------------------|
| `Invalid URL format: <string>` | Missing protocol scheme or illegal characters | Ensure URL begins with `https://`, `http://`, or target protocol. |
| `Protocol '<protocol>' is not allowed. Allowed: ...` | URL scheme not included in `AllowedProtocols` | Add protocol to allowlist or reject the incoming request. |
| `Data is required for URL validation` | Input value is `null` or `undefined` | Check upstream trigger/action node output. |
| `Cannot validate empty URL` | Empty string was passed (`""`) | Ensure form or API sends non-empty text. |

---

### Best Practices

- **Always Secure Webhooks:** Restrict `AllowedProtocols` to `["https"]` for all outgoing webhook calls to prevent man-in-the-middle data interception.
- **Use the `host` Output for Domain Whitelisting:** Connect the `host` output to a subsequent condition node to ensure URLs belong to your authorized organizational domains (e.g. ending in `.mycompany.com`).
- **Wire the Error Output:** Always connect the red `error` output to handle invalid links gracefully (e.g. return a friendly validation message or log an alert).

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related Nodes

- [Schema Validate](../schema-validate/en.md) — Comprehensive JSON Schema and field-level data validation
- [Validate Phone](../validate-phone/en.md) — Validate and standardize international phone numbers
- [HTML Sanitize](../html-sanitize/en.md) — Clean HTML content and eliminate XSS vulnerabilities
- [HTTP Request](../http-request/en.md) — Send web requests to validated endpoints
- [Webhook](../webhook/en.md) — Ingest inbound HTTP payloads containing URLs

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Description |
|:-------:|:----:|-------------|
| `1.0.0` | 2026-09-01 | Initial complete documentation for Validate URL action node with parameter guides, protocol allowlist rules, structured output references (`valid`, `url`, `protocol`, `host`), and real-world security patterns. |

<!-- /SECTION: changelog -->
