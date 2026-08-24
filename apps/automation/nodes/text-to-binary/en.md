---
node_id: "text-to-binary"
title: "Text to Binary"
description: "Convert text into binary data using a selected character encoding."
category: "Transformation (ETL)"
subcategory: "Encoding & Hashing"
version: "1.0.0"
language: "en"
last_updated: "2026-08-24"
author: "Fusion Team"
tags:
  - text
  - binary
  - encoding
  - transformation
related_nodes:
  - binary-to-text
  - encode-base64
  - function
---

<!-- SECTION: header -->

# Text to Binary

> **Category:** Transformation (ETL) | **Subcategory:** Encoding & Hashing | **Type:** Action Node

Convert text into binary data using a selected character encoding. Use the result when a downstream node or external service expects a buffer rather than a string.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->

## Overview

The **Text to Binary** node encodes text as binary data. It can use a configured value or text supplied by the incoming `input` port, making it suitable for file, HTTP, messaging, and data-transformation workflows.

### Key Features

- Converts plain text and string-compatible input to binary data
- Supports common Node.js character encodings
- Accepts a fixed value or dynamic data from a previous node
- Routes conversion failures to the `error` output

### Use Cases

- Prepare text for file creation or upload
- Build binary request bodies for APIs
- Convert generated text before passing it to a file or storage node
- Encode JSON, CSV, XML, or other textual payloads

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `data` | `string` | No | — | Text to encode. If omitted, the node reads text from the incoming `input`. |
| `encoding` | `enum` | No | `utf8` | Character encoding used to create the binary value. |

### Supported Encodings

Common values include:

- `utf8` — Recommended for modern text, JSON, CSV, Arabic, French, and emoji
- `ascii` — ASCII-only text
- `utf16le` — UTF-16 little-endian text
- `latin1` — Latin-1 or legacy Western European text
- `base64` — Treat the input as Base64 text when creating the buffer
- `hex` — Treat the input as hexadecimal text when creating the buffer

Use an encoding that matches the format expected by the downstream system. `utf8` is the safest default for ordinary text.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->

## Inputs & Outputs

### Input

| Port | Type | Description |
|------|------|-------------|
| `input` | `string` or `any` | Text to encode when `data` is not configured. Values are converted to text as needed. |

### Success Output

The `success` output contains the encoded binary value. A typical result is represented as:

```json
{
  "data": "<binary data>",
  "encoding": "utf8",
  "length": 5
}
```

For example, encoding `Hello` with `utf8` produces the bytes `48 65 6c 6c 6f`.

### Error Output

Conversion and validation failures are sent to `error`:

```json
{
  "data": null,
  "errorMessage": "Unable to convert text to binary",
  "errorCode": "TEXT_TO_BINARY_ERROR"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->

## Examples

### Basic Conversion

Configuration:

```json
{
  "data": "Hello",
  "encoding": "utf8"
}
```

The node returns the binary representation of `Hello`.

### Dynamic Input

Leave `data` empty and connect a previous node to the `input` port:

```json
{
  "$expr": "output",
  "node": "Function",
  "outputId": "success"
}
```

The incoming text is encoded using `utf8` unless another encoding is selected.

### JSON Payload

```json
{
  "data": "{\"name\":\"Fatima\",\"country\":\"Morocco\"}",
  "encoding": "utf8"
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->

## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Convert static and dynamic text to binary data
```

### Common Patterns

- **Static payload:** Manual Trigger → Text to Binary → Log
- **Dynamic payload:** Manual Trigger → Function → Text to Binary → Log
- **File preparation:** Text to Binary → Create File or upload node
- **Round trip:** Text to Binary → Binary to Text

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->

## Troubleshooting

### Missing text

**Cause:** Neither `data` nor a usable incoming `input` value was provided.

**Solution:** Configure `data` or connect a previous node to the `input` port.

### Unsupported encoding

**Cause:** The selected encoding is not supported by the runtime.

**Solution:** Use a supported encoding such as `utf8`, `ascii`, `utf16le`, `latin1`, `base64`, or `hex`.

### Incorrect characters downstream

**Cause:** The downstream system decoded the bytes using a different encoding.

**Solution:** Use the same encoding on both sides. For general text, use `utf8` consistently.

### Invalid Base64 or hexadecimal input

**Cause:** `base64` and `hex` interpret the input as encoded text and require valid characters and length.

**Solution:** Validate the source string, or use `utf8` when the value is ordinary text.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->

## Related

- [Binary to Text](./binary-to-text.md) — Decode binary data back into text
- [Encode Base64](./encode-base64.md) — Convert data to a Base64 string
- [Function](./function.md) — Generate or transform text before encoding

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->

## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-24 | Initial documentation |

<!-- /SECTION: changelog -->
